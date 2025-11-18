---
permalink: /posts/2024-09-16-LogRefPSA/
layout: single
title:  "PSA: UE_LOG_REF is Deprecated"
date:   2024-09-16 19:46:14 -0800
classes: wide
sidebar:
  nav: "docs"
categories: unreal
---

I was recently browsing through the UE 5.2 logging macros, and I noticed that `UE_LOG_REF` has been marked with "DO NOT USE":

```cpp
/**
 * DO NOT USE. A macro that logs a formatted message if the log category is active at the requested verbosity level.
 *
 * @note This does not trace the category correctly and will be deprecated in a future release.
 */
#define UE_LOG_REF(CategoryRef, Verbosity, Format, ...) \
```

Unfortunately, I don't see any indication that alternative macro/function will be provided. This means that if you are like me and used UE_LOG_REF for your custom logger, you'll need another solution.

AFAICT, there isn't a way to dynamically add the category to a UE_LOG at runtime (this was the whole point of UE_LOG_REF), so my workaround was to have my custom logger return a string and then to use that string from my custom logging macro:

```cpp
#define YOUR_CUSTOM_LOG(CategoryName, Verbosity, Format, ...) \
{                                                             \
    FString LogString = YourLogger::LogFn(/*your params*/)    \
    UE_LOG(CategoryName, Verbosity, TEXT("%s"), *LogString)   \
}
```