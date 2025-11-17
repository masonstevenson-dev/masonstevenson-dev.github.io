---
layout: post
title:  "UE Stats System Woes"
date:   2023-11-17 19:46:14 -0800
categories: 
---

# UE Stats System Woes

I'm frustrated.

All I needed was a nice way to keep track of errors that happen on tick. I wanted something that would count the errors and then display a warning message to the screen like:

**You have X errors: check the logs for more info**

Theoretically, UE already has a nice system set up for doing such a thing: the [UE Stats System](https://docs.unrealengine.com/5.3/en-US/unreal-engine-stats-system-overview/), which supports integer counters out of the box and has integration with the UE editor for displaying them. My plan was to just use the standard `INC_DWORD_STAT()` macro in place of logging wherever an error occurs, then write a little tickable object that would poll the stats system every so often to see if any of my error counters are greater than 0.

Unfortunately the UE stats system has several flaws that make it a huge pain for this particular use-case.



## The Problem(s)




#### 1 - The system for collecting stats is VERY tightly coupled with the system for displaying them.

In order to fetch stats from the stats system in c++, you need to extract the stats data from FLatestGameThreadStatsData:

```cpp
FGameThreadStatsData* ViewData = FLatestGameThreadStatsData::Get().Latest;
if (!ViewData || !ViewData->bRenderStats)
{
    return;
}

for (int32 GroupIndex = 0; GroupIndex < ViewData.ActiveStatGroups.Num(); ++GroupIndex)
{
    FString StatGroupName = ViewData.GroupNames[GroupIndex].ToString();

    if (!StatGroupName.Equals(TEXT("STATGROUP_YourStatGroup")))
    {
        continue;
    }

    const FActiveStatGroupInfo& ErrorStats = ViewData.ActiveStatGroups[GroupIndex];
    for (const FComplexStatMessage& ErrorStat : ErrorStats.CountersAggregate)
    {
        if (ErrorStat.NameAndInfo.GetField<EStatDataType>() == EStatDataType::ST_double)
        {
            double StatValue = ErrorStat.GetValue_double(EComplexStatField::IncMax);
            // do something with the stat value...
        }
        else if (ErrorStat.NameAndInfo.GetField<EStatDataType>() == EStatDataType::ST_int64)
        {
            int64 StatValue = ErrorStat.GetValue_int64(EComplexStatField::IncMax);
            // do something with the stat value...
        }
    }
}
```

Here's the problem though: STATGROUP_YourStatGroup wont even be collected unless it is "enabled". There are two ways to enable your stat. You can enable From the UE editor:

![](/assets/images/01_stat_select.png) 

But then the stat UI ends up taking up a huge portion of the screen:

![](/assets/images/02_stat_fills_entire_screen.png) 

Alternatively you can enable via the console. If you use the -nodisplay flag, you can get the system to collect your stat without actually displaying anything: `stat YourStat -nodisplay`. However, I want my stats to *always* be collected. I figured maybe I could just write some code that fires on world create to call this stat command and force the stat on:

```cpp
// put this somewhere that will trigger it when your game is running

// fetch or pass in the current UWorld
UWorld* CurrentWorld = GetWorld();

GEngine->Exec(CurrentWorld, TEXT("stat YourStat -nodisplay"));
```

However, this leads us to our next problem...

#### 2 - The 'stat' has no on/off switch

Annoyingly, calling `stat YourStat` will always *toggle* it. There is no way to say something like `stat YourStat -enable` to force it always on. This means that if your stat happens to be enabled before you press the play button, calling `stat YourStat -nodisplay` will incorrectly turn it off instead of keeping it on.

A (hacky) workaround for this is to always turn off all the stats before turning your stat on:

```cpp
// put this somewhere that will trigger it when your game is running

// fetch or pass in the current UWorld
UWorld* CurrentWorld = GetWorld();

// Flush all stats so we can guarentee that YourStat will be enabled.
GEngine->Exec(CurrentWorld, TEXT("stat none"));
GEngine->Exec(CurrentWorld, TEXT("stat YourStat -nodisplay"));
```

At this point, I was already feeling pretty unhappy with the way things were going, but what finally killed my desire to keep pushing for using the stats system was the third and final issue:

#### 3 - Enabling ANY other stat completely overrides your -nodisplay flag set earlier!

Title says it all. Want to  check the FPS real quick with `stat FPS`? BAM! Enjoy having whatever stats you were collecting with `stat XYZ -nodisplay` pop up and fill your screen.

I briefly toyed around with the idea of making my own custom stat command just to get around this issue:

```cpp
FAutoConsoleCommandWithWorldAndArgs GAnankeLogScreenShowCategoriesCmd(
	TEXT("statfix"),
	TEXT("works like the regular stat command, but re-hides MyStat after."),
	FConsoleCommandWithWorldAndArgsDelegate::CreateStatic(
		[](const TArray<FString>& Args, UWorld* World)
		{
            FString StatCommand = FString("stat");
            for (Fstring Arg : Args)
            {
                StatCommand += FString(" ") + Arg;
            }
            
			GEngine->Exec(World, StatCommand);
            
            // If MyStat is enabled, this will hide it. If it is disabled, nothing will happen.
            GEngine->Exec(CurrentWorld, TEXT("stat MyStat -nodisplay"));
            GEngine->Exec(CurrentWorld, TEXT("stat MyStat -nodisplay"));
		}
	)
);
```

At this point, I felt that I was approaching radioactive levels of jank, so I decided to just throw out the whole stat logging dream.

## What I did instead

Instead, I decided to just implement my own periodic logger. I have a nice little macro I can call that pushes logs out to the logger:

```cpp
if (SomeErrorOccured)
{
    // Log this at most every three seconnds.
    ANANKE_LOG_PERIODIC(Error, TEXT("SomeError has occured."), 3.0)
}
```

When called, the macro reaches out to my logging subsystem (of type UEngineSubsystem) which increments a counter for this particular log as well as marks it as pending. Logs are identified via a hash of the file name and line number in which they appeared. A separate UTickableWorldSubsystem is set up to poll the logging system and find all pending logs that are meet the time requirements set by the macro and pushes them out as regular UE_LOGs.

The final result log looks something like this:

```
[2023.01.01-00.00.00:000][  0] LogAnankePeriodic: some_file.cpp:11 PERIODIC_LOG(count=37): SomeError has occured.
```



Is this thread safe? no. Is this performant? Probably not.  But it does what I want, and I will compile it out of shipping builds anyway. 


