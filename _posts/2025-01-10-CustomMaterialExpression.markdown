---
layout: post
title:  "Defining Custom Material Nodes with C++ and HLSL"
date:   2025-01-10 19:46:14 -0800
categories: 
---

Ever wonder how you can create your own custom material expression nodes in Unreal? Not material functions, but the actual green-border material expression nodes (like add, subtract, lerp, etc)?

![](/assets/images/01_lerp.png) 


It turns out there's a (fairly straightforward) way to define these in wholly C++, and a (slightly less straightforward) way to define these as HLSL functions and then call them from C++.



# How to set up UMaterialExpressions

The C++ class Unreal uses to define material expression nodes is **UMaterialExpression** (see Engine/Source/Runtime/Engine/Classes/Materials/MaterialExpression.h). To create your own material, you simply need to subclass this and implement a few functions:

* **Your class constructor**
  * This is where you can set up the menu category that you want your node to appear in when you right click in the material graph editor to pull up the material node menu.
* **GetCaption()**
  * This controls the text label that appears on the physical node itself.
* **GetCreationName()**
  * This controls the text that appears when you search for your node in the material node menu.
* **Compile()**
  * This is the function that actually contains the node logic. Note that this function is supposedly scheduled to be replaced by GenerateHLSLExpression() at some point, but it is unclear (as of UE5.5) what the status of this migration is. See **Compile() vs Generate HLSLExpression()** below for more details
* **GenerateHLSLExpression()**
  * The "new" way to define node logic.



In addition to the these functions, you must also add a UPROPERTY() of type FExpressionInput for each input pin you want on your node. You can optionally also add a UPROPERTY() for the default value of each input pin. You can also add other UPROPERTY() values that can be used to change the node's behavior.

```c++
UCLASS(Abstract)
class UYourCustomMaterialNode : public UMaterialExpression
{
	GENERATED_BODY()

public:
	UYourCustomMaterialNode(const FObjectInitializer& Initializer);

#if WITH_EDITOR
	//~ Begin UMaterialExpression Interface
	virtual int32 Compile(class FMaterialCompiler* Compiler, int32 OutputIndex) override;
    virtual bool GenerateHLSLExpression(
        FMaterialHLSLGenerator& Generator,
        UE::HLSLTree::FScope& Scope,
        int32 OutputIndex,
        UE::HLSLTree::FExpression const*& OutExpression
    ) const override;
	
	virtual void GetCaption(TArray<FString>& OutCaptions) const override;
	virtual FText GetCreationName() const override { return FText::FromString(TEXT("My Node")); }
	virtual void GetExpressionToolTip(TArray<FString>& OutToolTip) override;
	//~ End UMaterialExpression Interface
#endif // WITH_EDITOR

	UPROPERTY(meta = (RequiredInput = "true", ToolTip = "InputA is required for the material to compile."))
	FExpressionInput InputA;
    
    UPROPERTY(meta = (RequiredInput = "false", ToolTip = "Optional. If not connected, use DefaultInputB."))
	FExpressionInput InputB;
    
    UPROPERTY(EditAnywhere, Category=MaterialExpressionMyNode, meta=(OverridingInputProperty = "InputB"))
	float DefaultInputB;
    
    UPROPERTY(EditAnywhere, Category=MaterialExpressionMyNode)
    bool bNegateResult;
};
```



In the constructor, set up your menu category and assign default values:

```c++
UYourCustomMaterialNode::UYourCustomMaterialNode(const FObjectInitializer& Initializer): Super(Initializer)
{
	struct FConstructorStatics
	{
		FText YourCategory;
		FConstructorStatics(): YourCategory(LOCTEXT( "MyNodeCategory", "My Custom Nodes" ))
		{
		}
	};
	static FConstructorStatics ConstructorStatics;

#if WITH_EDITORONLY_DATA
	MenuCategories.Add(ConstructorStatics.YourCategory);
#endif

	DefaultInputA = 100.0f;
	DefaultInputB = 1.0f;
}
```



In the Compile() function, you can use the provided Compiler object to evaluate the inputs as well as perform different common operations. You can perform a check on GetTracedInput().Expression for each of your optional pins and assign your default values if the expression is missing (i.e. the pin is not connected in the material graph). You can also check your custom properties here to alter the behavior of your node.

```cpp
#if WITH_EDITOR
int32 UYourCustomMaterialNode::Compile(class FMaterialCompiler* Compiler, int32 OutputIndex)
{
	// Adds A and B together, and optionally negates the result.

	int32 InputAResultID = InputA.Compile(Compiler);
	int32 InputBResultID = InputB.GetTracedInput().Expression ? InputB.Compile(Compiler) : Compiler->Constant(DefaultInputB);

	int32 AddResultID = Compiler->Add(Arg1, Arg2);
	
	if (!bNegateResult)
	{
		return AddResultID;
	}
	
	return Compiler->Mul(Compiler->Constant(-1.0f), AddResultID);
}
#endif // WITH_EDITOR
```



In the GenerateHLSLExpression() function, you use the provided Generator instead of the Compiler:

```c++
#if WITH_EDITOR
bool UYourCustomMaterialNode:GenerateHLSLExpression(
    FMaterialHLSLGenerator& Generator,
    UE::HLSLTree::FScope& Scope,
    int32 OutputIndex,
    UE::HLSLTree::FExpression const*& OutExpression
) const override;
{
	// Adds A and B together, and optionally negates the result.
	const UE::HLSLTree::FExpression* InputAExpression = InputA.AcquireHLSLExpression(Generator, Scope);
	const UE::HLSLTree::FExpression* InputBExpression = InputB.AcquireHLSLExpressionOrConstant(Generator, Scope, DefaultInputB);
	if (!InputAExpression || !InputBExpression)
	{
		return false;
	}
    
	const UE::HLSLTree::FExpression* OutExpression = Generator.GetTree().NewAdd(InputAExpression, InputAExpression);
    if (bNegateResult)
    {
        OutExpression = Generator.GetTree().NewMul(Generator.NewConstant(-1.0f), OutExpression);
    }
    
	return true;
}
#endif // WITH_EDITOR
```



## Compile() vs GenerateHLSLExpression()

If you inspect the UE source code, you'll find references to GenerateHLSLExpression() being the replacement for Compile(). It's hard to find concreate information about what makes GenerateHLSLExpression() better than Compile, but my best guess is that the new HLSL generation system is basically a more advanced version of an existing feature in the UE material graph called constant folding, which is an optimization where UE can collapse certain node groups if it is known at compile time that the result from that node group will not change. So for example if you have a group of nodes where you add 3+4 and then multiply by 2, UE will just collapse that cluster of nodes down to a constant value of 14.

![](/assets/images/02_constant_folding_simple_example.png) 


I *think* the new HLSL generator can basically do this also, but it can evaluate node clusters at runtime (instead of just compile time) and then effectively prune them by caching the results if the result is uniform. Don't quote me though.



Regardless, the current state of the new HLSL generator seems to be up in the air. The system has been in development since at least 2021, and it is still tagged as "WIP". Furthermore, the console variable that enables this feature (r.MaterialEnableNewHLSLGenerator) seems to be turned off by default for all UE users. Ultimately, you have two options:

1) Just implement Compile() for now, with the understanding that your material node might stop working if Epic ever decides to switch everybody over to the new HLSL generator.
2) Implement both Compile() and GenerateHLSLExpression() now just to be safe, with the understanding that GenerateHLSLExpression() isn't used right now and might be unstable.



For my projects, I'm choosing to go with #1. I can always go back in and add the missing generator function later if it becomes necessary.



# How to call HLSL from your UMaterialExpression

One thing you might want to do if you have custom HLSL shader code defined in .ush or .usf files is to just call your HLSL directly from your custom material node. It turns out there is way to do this, by creating a wrapper class around the UMaterialExpressionCustom class, which is UMaterialExpression subclass that is responsible for the "Custom" node:

![](/assets/images/03_default_custom_node.png) 


First, define your class as normal, but add a new pointer for a UMaterialExpressionCustom. Also add GetInternalExpression() function so you can lazy-load the object.

```c++
UCLASS(Abstract)
class UYourCustomMaterialNode : public UMaterialExpression
{
	GENERATED_BODY()

public:
	UYourCustomMaterialNode(const FObjectInitializer& Initializer);

#if WITH_EDITOR
	//~ Begin UMaterialExpression Interface
	virtual int32 Compile(class FMaterialCompiler* Compiler, int32 OutputIndex) override;
    virtual bool GenerateHLSLExpression(
        FMaterialHLSLGenerator& Generator,
        UE::HLSLTree::FScope& Scope,
        int32 OutputIndex,
        UE::HLSLTree::FExpression const*& OutExpression
    ) const override;
	
	virtual void GetCaption(TArray<FString>& OutCaptions) const override;
	virtual FText GetCreationName() const override { return FText::FromString(TEXT("My Node")); }
	virtual void GetExpressionToolTip(TArray<FString>& OutToolTip) override;
	//~ End UMaterialExpression Interface
#endif // WITH_EDITOR

	UPROPERTY(meta = (RequiredInput = "true", ToolTip = "InputA is required for the material to compile."))
	FExpressionInput InputA;
    
    UPROPERTY(meta = (RequiredInput = "false", ToolTip = "Optional. If not connected, use DefaultInputB."))
	FExpressionInput InputB;
    
    UPROPERTY(EditAnywhere, Category=MaterialExpressionMyNode, meta=(OverridingInputProperty = "InputB"))
	float DefaultInputB;
    
    UPROPERTY(EditAnywhere, Category=MaterialExpressionMyNode)
    bool bNegateResult;
    
    // NEW STUFF- to support HLSL function call
private:
#if WITH_EDITOR
    UMaterialExpressionCustom* GetInternalExpression();
#endif // WITH_EDITOR
    
	UPROPERTY()
	TObjectPtr<UMaterialExpressionCustom> CustomExpression = nullptr;
};
```



Implement the constructor as normal. In your GetInternalExpression() function, lazy-load the object and then set the following parameters on it:

* **Expression->Inputs**:
  * Defines the name of each input you want to use
* **Expression->OutputType**:
  * Defines what the output of your node will be
* **Expression->IncludeFilePaths**:
  * The paths to the shader files you want to include
  * Quick reminder: If you have your .ush files defined in a plugin shader directory like "/SomePlugin/Shaders/YourFile.ush", you need to go to your plugin module and call AddShaderSourceDirectoryMapping() to map the filepath.
* **Expression->Code**: If the node always has the same behavior, add the code here. Otherwise, you can dynamically set the code inside the Compile/Generate function.



```cpp
#if WITH_EDITOR
UMaterialExpressionCustom* UYourCustomMaterialNode::GetInternalExpression()
{
	if (CustomExpression)
	{
		return CustomExpression;
	}

	CustomExpression = NewObject<UMaterialExpressionCustom>();
	CustomExpression->Inputs[0].InputName = TEXT("InputA"); // the first input is already added
	CustomExpression->Inputs.Add({ TEXT("InputB") });
	CustomExpression->OutputType = ECustomMaterialOutputType::CMOT_Float1;
	CustomExpression->IncludeFilePaths.Add("/YourPlugin/Shaders/YourShaderFile.ush");
	return CustomExpression;
}
#endif // WITH_EDITOR
```



In the Compile function, fetch your wrapped UMaterialExpressionCustom and feed it to Compiler->CustomExpression

```c++
#if WITH_EDITOR
UMaterialExpressionCustom* UYourCustomMaterialNode::Compile(class FMaterialCompiler* Compiler, int32 OutputIndex)
{
	UMaterialExpressionCustom* InternalExpression = GetInternalExpression();
	if (!InternalExpression)
	{
		return Compiler->Errorf(TEXT("Internal expression is null."));
	}

	if (bNegateResult)
	{
        // Silly example, because you could just use the built-in HLSL add and multiply, but you get the idea.
		InternalExpression->Code =
			TEXT(R"(
				return YourMultFunction(YourAddFunction(InputA, InputB), -1.0);
			)");
	}
	else
	{
		InternalExpression->Code =
			TEXT(R"(
				return YourAddFunction(InputA, InputB);
			)");
	}

	// Just to be safe, clear out the InternalExpression input. This should only be used by GenerateHLSLExpression
	// Remove this if not useing GenerateHLSLExpression.
	InternalExpression->Inputs[0].Input = FExpressionInput();
	InternalExpression->Inputs[1].Input = FExpressionInput();

	int32 InputAResultID = InputA.Compile(Compiler);
	int32 InputBResultID = InputB.GetTracedInput().Expression ? InputB.Compile(Compiler) : Compiler->Constant(DefaultInputB);
	TArray<int32> Inputs{ InputAResultID, InputBResultID };
	
	return Compiler->CustomExpression(InternalExpression, /*OutputIndex=*/0, Inputs);
}
#endif // WITH_EDITOR
```



In GenerateHLSLExpression, you pretty much do the same thing, but you then call the internal GenerateHLSLExpression() on your wrapped UMaterialExpressionCustom class. Note, I have not personally tested the code below, but I believe it should work.

```cpp
#if WITH_EDITOR
bool UYourCustomMaterialNode::GenerateHLSLExpression(
    FMaterialHLSLGenerator& Generator,
    UE::HLSLTree::FScope& Scope,
    int32 OutputIndex,
    UE::HLSLTree::FExpression const*& OutExpression
) const
{
	UMaterialExpressionCustom* InternalExpression = GetInternalExpression();
	if (!InternalExpression)
	{
		return Generator.Errorf(TEXT("Internal expression is null."));
	}

	if (bNegateResult)
	{
        // Silly example, because you could just use the built-in HLSL add and multiply, but you get the idea.
		InternalExpression->Code =
			TEXT(R"(
				return YourMultFunction(YourAddFunction(InputA, InputB), -1.0);
			)");
	}
	else
	{
		InternalExpression->Code =
			TEXT(R"(
				return YourAddFunction(InputA, InputB);
			)");
	}

	InternalExpression->Inputs[0].Input = InputA;
	InternalExpression->Inputs[1].Input = InputB;

	return InternalExpression->GenerateHLSLExpression(Generator, Scope, OutputIndex, OutExpression);
}
#endif // WITH_EDITOR
```



# Additional Resources

* [Unreal Docs - Custom Expressions](https://dev.epicgames.com/documentation/en-us/unreal-engine/custom-expressions?application_version=4.27)
* [Unreal Docs - Overview of Shaders in Plugins](https://dev.epicgames.com/documentation/en-us/unreal-engine/overview-of-shaders-in-plugins-unreal-engine)
* [Tech Art Aid - Shader Optimization Tutorial](https://www.youtube.com/watch?v=y0QASid1v8w)

