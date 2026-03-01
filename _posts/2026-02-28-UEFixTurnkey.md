---
permalink: /posts/2026-02-28-UEFixTurnkey/
layout: single
title: "How to Fix \"Turnkey returned an error, code 1\" (UE Source Builds)"
date: 2026-02-28 00:00:00 -0800
show_date: true
classes: wide
sidebar:
  nav: "docs"
categories: unreal
author_profile: true
---



I recently had an issue where I was trying to package my Unreal Engine project (5.6, source build), and I ran into a weird error:



When I tried to package my Linux server, I got a popup stating:

> "The SDK for Linux is not installed properly, which is needed to generate data. Check the SDK section of the Launch On menu in the main toolbar to update SDK."



When I went to **Platforms->Linux->SDK Management**, there was an error:

>"Turnkey returned an error, code 1 (See log)"



In the UE output log:

> AutomationTool encountered an error before a log file was set, see 'D:\Unreal\EngineSource\5.6\Engine\Programs\AutomationTool\Saved\Logs\ErrorLog.txt' for more details
> AutomationTool executed for 0h 0m 2s
> AutomationTool exiting with ExitCode=1 (Error_Unknown)
> BUILD FAILED



Hmm, very strange. When I went to that ErrorLog.txt file, nothing seemed to be amiss. 



## The Fix

The root cause ended up being a problem with **Package 'Magick.NET-Q16-HDRI-AnyCPU'**. For some reason, this package becomes invalidated every few weeks due to having "a known moderate security vulnerability". I actually already encountered this once when I first tried to build the engine source, but apparently the UE AutomationTool (and therefore the UE packaging system) breaks pretty much every time that Magick.NET package updates. Frustratingly, whatever error this causes never seems to make it into the logs.



First, to test if this is actually your problem: open up the solution for your UE source build and try to rebuild the "AutomationTool" target. You should get a bunch of **Warning as Error** messages regarding **Package 'Magick.NET-Q16-HDRI-AnyCPU'**.



If you see these errors, check [the nuget.org page for Magick.NET](https://www.nuget.org/packages/Magick.NET-Q16-HDRI-AnyCPU#versions-body-tab) to see what the latest version is, then go and update the version for that package in the following engine config files:

* **AutomationTool.csproj**
* **AutomationUtils.Automation.csproj**
* **Gauntlet.Automation.csproj**



For reference, here's an [example commit](https://github.com/EpicGames/UnrealEngine/commit/360060ea33d92fe562e256456bb51197788119f0) of a UE dev doing just that.



Rebuild the AutomationTool target, restart your UE editor- and you should be able to package stuff again.