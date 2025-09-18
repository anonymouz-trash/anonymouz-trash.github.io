---
layout: post
title:  "Windows games on Android"
categories: [Android Gaming]
tags: [android,gaming,windows,winlator,gamenative,gamehub,adreno,turnip]
image:
  path: /assets/img/2025-09-18-windows-games-on-android.jpg
last_modified_at: 2025-09-18 06:00:00 +0100
---

## Introduction
I recently bought myself a new gaming handheld. After long waiting and struggling I decided to buy the faboulus mid-range device **Retroid Pocket Flip 2**.
I don't have any other device to test or compare settings or tweaks I found out and describe in this article. Please keep that in mind.

Specs:
| Part | Value |
| CPU | Qualcomm Snapdragon 865 |
| Cores | 1*A77@2.8G 3*A77@2.4G 4*A55@1.8G |
| Arch | arm64 |
| RAM | 8 GB LPDDR4x@2133MHz |
| GPU | Adreno 650 |
| Int. Storage | 128 GB |
| Display | 5.5" AMOLED, 1080p@60fps |

## Important notice
I'm not responsible for anything that is not working after reading through this guide. It is important for you to understand that the entire subject area is **highly experimental**.
So therefor using an android gaming handheld as daily driver for playing windows games shouldn't be your first goal. It's more like a nerdy tech thing and a prove of concept.

As with everything in life it is all a "hit & miss" or "trail & error". Don't give up to fast! The developers behind all mentioned projects are doing a very great job! Progress guranteed!

## Recommended prerequisites
I encourage you to download the following additional software or drivers and bookmark their sources. You'll need them every now and then.

### Adreno GPU drivers (Snapdragon only)
* AdrenoToolsDrivers: This is an approach for downloading K11MCH1's GPU drivers with automatic detection of which is the best for your chipset.
Source: [https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/fetcher_v1.2](https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/fetcher_v1.2)
* Mesa Turnip driver v24.3.0 - Revision 9v2: I suggest this driver as best one for all cases, like Winlator cmod or Eden
Source: [https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/v24.3.0_r9](https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/v24.3.0_r9)
* turnip_mrpurple-T19-toasted.adpkg: This is the driver automatically choosed by AdrenoToolsDrivers. I think it is not as good as the above because of artifacts and graphic glitches every now and then.
Source: [https://github.com/MrPurple666/purple-turnip/releases/tag/vturnip_mrpurple-T19-toasted.adpkg](https://github.com/MrPurple666/purple-turnip/releases/tag/vturnip_mrpurple-T19-toasted.adpkg)

> I for myself search through the release pages in GitHub with the keywords: 650, 6xx, A650 or A6xx. ;-)
{: .prompt-info}

### Windows emulators
* Winlator by BrunoSX: This is the ground root of all these amazing projects. Without it any other project wouldn't be possible without it.
Source: [https://github.com/brunodev85/winlator](https://github.com/brunodev85/winlator)
* Winlator Cmod by coffincolors: Best Winlator fork in my opinion.
Source: [https://github.com/coffincolors/winlator](https://github.com/coffincolors/winlator)
* GameNative: Open Source project by Utkarsh Dalal to create a native Steam gaming experience. I would try this one before GameHub.
Source: [https://github.com/utkarshdalal/GameNative](https://github.com/utkarshdalal/GameNative)
* GameHub: Another Fork of Winlator, but with the goal to create a nearly native Steam gaming experience. Worth a try. It is made by the creators of the GameSir controllers.
Source: [https://gamehub.xiaoji.com/](https://gamehub.xiaoji.com/)

### Useful sites
* [Winlator101 - comprehensive Winlator guide](https://github.com/K11MCH1/Winlator101)
* [Winlator101 - compatibility list with settings by K11MCH1](https://github.com/K11MCH1/Winlator101/issues?q=is%3Aissue%20label%3Aplayable&page=1)
* [EmuReady - combatibility database](https://www.emuready.com/)
* [Retroid Pocket Starter Guide by RetroGameCorps](https://retrogamecorps.com/2022/01/16/retroid-pocket-2-starter-guide/)

## Winlator CMOD
All I describe in this guide is, in any way, useable in the other apps (forks) of Winlator.

**WIP**