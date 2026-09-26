---
layout: post
title:  "Windows games on Android"
categories: [Android Gaming]
tags: [android,gaming,windows,winlator,gamenative,gamehub,adreno,turnip]
image:
  path: /assets/img/2025-09-18-windows-games-on-android.jpg
last_modified_at: 2025-09-19 10:31:00 +0100
---

## Introduction
I recently bought myself a new gaming handheld. After a long wait and much deliberation, I decided to buy the fabulous mid-range device **Retroid Pocket Flip 2**.
I don't have any other device to test or compare settings or tweaks I found out and describe in this article. Please keep that in mind.

Technical details:

| Part | Value |
| --- | --- |
| CPU | Qualcomm Snapdragon 865 |
| Cores | 1x A77@2.8G; 3x A77@2.4G; 4x A55@1.8G |
| Arch | arm64 |
| RAM | 8 GB LPDDR4x@2133MHz |
| GPU | Adreno 650 |
| Int. Storage | 128 GB |
| Display | 5.5" AMOLED, 1080p@60fps |

## Important notice

> It is important for you to understand that the entire subject area is **highly experimental**. So, using an Android gaming handheld as a daily driver for playing Windows games shouldn't be your first goal. It's more like a nerdy tech thing and a proof of concept. As with everything in life, it's all "hit & miss" or "trial & error". Don't give up too fast! The developers behind all the mentioned projects are doing a great job! Progress guaranteed!
{: .prompt-warning}

## Recommended prerequisites
I encourage you to download the following additional software or drivers and bookmark their sources. You'll need them every now and then.

### Adreno GPU drivers (Snapdragon only)
* AdrenoToolsDrivers: This is an approach for downloading K11MCH1's GPU drivers with automatic detection of which is the best for your chipset.
Source: [https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/fetcher_v1.2](https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/fetcher_v1.2)
* Mesa Turnip driver v24.3.0 - Revision 9v2: I suggest this driver as the best one for all cases, like Winlator Cmod or Eden
Source: [https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/v24.3.0_r9](https://github.com/K11MCH1/AdrenoToolsDrivers/releases/tag/v24.3.0_r9)
* turnip_mrpurple-T19-toasted.adpkg: This is the driver automatically chosen by AdrenoToolsDrivers. I think it is not as good as the one above, because of artifacts and graphic glitches every now and then.
Source: [https://github.com/MrPurple666/purple-turnip/releases/tag/vturnip_mrpurple-T19-toasted.adpkg](https://github.com/MrPurple666/purple-turnip/releases/tag/vturnip_mrpurple-T19-toasted.adpkg)

> I myself search through the release pages on GitHub with the keywords: 650, 6xx, A650 or A6xx. ;-)
{: .prompt-info}

### Windows emulators
* **Winlator** by BrunoSX: This is the root of all these amazing projects. Without it, none of the other projects would be possible.
Source: [https://github.com/brunodev85/winlator](https://github.com/brunodev85/winlator)
* **Winlator Cmod** by coffincolors: Best Winlator fork in my opinion.
Source: [https://github.com/coffincolors/winlator](https://github.com/coffincolors/winlator)
* **GameNative** by Utkarsh Dalal: Open Source project to create a native Steam gaming experience. I would try this one before GameHub.
Source: [https://github.com/utkarshdalal/GameNative](https://github.com/utkarshdalal/GameNative)
* **GameHub** by GameSir company: Another fork of Winlator, but with the goal of creating a nearly native Steam gaming experience. Worth a try. It is made by the creators of the GameSir controllers.
Source: [https://gamehub.xiaoji.com/](https://gamehub.xiaoji.com/)

### Useful sites
* [Winlator101 - comprehensive Winlator guide](https://github.com/K11MCH1/Winlator101)
* [Winlator101 - compatibility list with settings by K11MCH1](https://github.com/K11MCH1/Winlator101/issues?q=is%3Aissue%20label%3Aplayable&page=1)
* [EmuReady - combatibility database](https://www.emuready.com/)
* [Retroid Pocket Starter Guide by RetroGameCorps](https://retrogamecorps.com/2022/01/16/retroid-pocket-2-starter-guide/)

## Winlator Cmod
Everything I describe in this guide is, in any case, usable in the other apps (forks) of Winlator. The latest version used, as of writing this guide, is/was v13.1.1.

### Overview

First, Winlator must initialize. Let's let it do its thing.
![winlator-init](/assets/img/winlator-init.png)

After everything has finished, the burger menu on the top left should be available. Tap it.
![winlator-burger-menu](/assets/img/winlator-burger-menu.png)

| menu item | description |
| --- | --- |
| Shortcuts | It is possible to create a shortcut for each application within a container. This is useful when you have multiple applications that need different container settings or controller bindings. It can all be managed with the Shortcuts. |
| Containers | The home of Winlator. Windows container creation. In most situations you just need one container or one per type. (Box64 or Arm64 (FEX)) |
| Controller Manager | Controller *magic* happens here. You are able to assign the Retroid *Onboard* controller for automatic detection in Steam. |
| Input Controls | Within this menu you map each Keyboard/Mouse/Gamepad key to a controller button. Useful for games without gamepad support. |
| Saves | More to that later. Not important for now. |
| Box64 RCFile | More to that later. Not important for now. RC files in linux usually load environmental variables and such things. More to that later. The important ones are already on by default. |
| Contents | Here you have the option to load other versions of programs or addons like DXVK, WineD3D and so on. Not important for now. |
| Settings | This is the place of Box 64 presets and other things like dark mode for the application itself. |

### Shortcuts
In this section, I will add tweaks per game as soon as I find them out.

#### Steam
You can install Steam as you normally would on a Windows PC. If you use an SD card, set it up in your container as described in the Containers section. Then you don't have to struggle with managing your Steam settings and library.

| menu | setting | value | description |
| --- | --- | --- | --- |
| interface | GPU Acceleration | off | I noticed a more responsive / reactive interface. |
| interface | News-Window at App-Start | off | Loading this takes time. |

When you edit the shortcut you'll have the same options as you would configure the container itself. Set the following:

| item | variable | value | description |
| --- | --- | --- | --- |
| Advanced | Box64 Preset | Unity / Non-Unity | Select / Set the right preset for the game you want to play. |
| Advanced | Input Controls | Profile-Name | If you created a controller profile select it here. |
| Advanced | Exec Arguments | Box below | If you notice that Steam won't start after installation or per shortcut, this is a workaround. |

```shell
-vgui -nocrashmonitor -noshaders -no-shared-textures -cef-single-process -cef-in-process-gpu -cef-disable-sandbox -disable-winh264 -no-cef-sandbox -vrdisable -cef-disable-breakpad -cef-disable-gpu -no-dwrite -no-gameoverlayrenderer -noverifyfiles -nobootstrapupdate -skipinitialbootstrap -norepairfiles -overridepackageurl
```
> Better to copy & paste ^^


#### NFS Most Wanted Black Edition (2005)

If you want to use Widescreen mods which use a modified `dinput8.dll` to load the mods you **must** define that as an environment variable. This procedure also works with Wine on normal desktop PCs.

| variable | value |
| --- | --- |
| WINEDLLOVERRIDES | dinput8=n,b |

### Containers
In this section, I'll try to explain the important settings as best I can. 
![winlator-container-wrapper](/assets/img/winlator-container-wrapper.png)
Set your preferred display resolution. I recommend leaving it at `1280x720`.
![winlator-container-wrapper-gpu-driver](/assets/img/winlator-container-wrapper-gpu-driver.png)

#### GPU driver
You can choose between `Wrapper` and `Wrapper-v2`. I didn't notice any differences between those, but next to it, tap the `gear` icon and choose your desired GPU driver.
If you have a Snapdragon CPU with an Adreno GPU, leave this as is; for any other CPU vendor, you must select `System`.
I think the `v762` and `v805` are for Snapdragon 8 Gen x and Snapdragon 8 Elite.
![winlator-container-wrapper-dxvk](/assets/img/winlator-container-wrapper-dxvk.png)
I suggest using the latest version available. Besides that, the suffix `gplasync` is important. Turn both async switches `on`.

> Tap the `question mark` icon next to DX-Wrapper to get a list of all options and their meanings.
{: .prompt-tip}

#### Audio driver
Always use `ALSA-Reflector`, because it prevents audio from breaking during gameplay.
![winlator-container-audio](/assets/img/winlator-container-audio.png)
More on that [here](https://github.com/coffincolors/winlator/releases/tag/cmod_v13).

#### Wine Configuration
![winlator-container-winecfg](/assets/img/winlator-container-winecfg.png)
The only things you may adjust are `Theme` and `Video Memory Size`.

| variable | value | description |
| --- | --- | --- |
| Theme | light | Default value, optional because there are no windows when starting through Shortcuts |
| Renderer | gl | Default value, as for now, leave it as is, because Vulkan is broken. The developers are aware of that. |
| Video Memory Size | 2048 | Default value |

#### Environment Variables
You can leave everything here as is. This is the section for customizing things per shortcut.
![winlator-container-env-var](/assets/img/winlator-container-env-var.png)

| variable | value | description |
| --- | --- | --- |
| DXVK_HUD | devinfo, fps, frametimes, gpuload, version, api | Default value, you can safely remove it if you don't want a performance monitor displayed permanently. |
| MESA_EXTENSION_MAX_YEAR | 2003 | optional, if older games don't open try this |
| MANGOHUD | off | default value, if you want you can enable Mangohud. |

#### Drives
It is recommended to also use an SD card as external storage. Therefore, you need to add a new mount point and give it the path to the SD card. Unfortunately, the path is not easy to find with stock apps.
![winlator-container-drives](/assets/img/winlator-container-drives.png)
Some file managers from Google Play or F-Droid are able to retrieve the `ID` of the SD card, which you'll need. It should look like in the picture. You can also retrieve it from your PC when the card is inserted, or if you use Termux, just type `df -h` in the terminal and copy/paste the path that looks like the one shown in the picture above.

#### Advanced
![winlator-container-advanced](/assets/img/winlator-container-advanced.png)
In this section you choose your previously created `Box64-Preset`, the `RC file` and maybe the `Startup Selection`.

> In `Shortcuts` sub-menu you have additional options like `Input Controls Profiles` and `Exec Arguments`.
{: .prompt-tip}

### Controller Manager
First, assign the Retroid Pocket controller as Player 1.
![winlator-burger-menu](/assets/img/winlator-burger-menu.png)
The result should look something like this. Btw you can also add a second external controller as Player 2. This is great if you use your handheld as console connected to an external display.
![winlator-controller-manager-assign](/assets/img/winlator-controller-manager-assign.png)

### Input Controls
In this sub-menu you can map keybindings and create (and also export) profiles for games/applications that don't have native gamepad support. You can set keybindings by tapping on your controller at the bottom.
![winlator-input-controls](/assets/img/winlator-input-controls.png)
This will open another sub-menu where you press each button you want to map and configure it afterwards.
![winlator-input-controls-bindings](/assets/img/winlator-input-controls-bindings.png)

### Settings (Box64)
The most important settings here are the Box64 presets.
![winlator-box64-presets](/assets/img/winlator-box64-presets.png)

Tap on the `+`-symbol and create three profiles called `Unity (MonoBleedingEdge)`, `Unity (GameAssembly)` and `Non-Unity`.
This is significant for running games based on Unity and the rest.
You can look up a short description and possible values in ptitSeb's [Box64 GitHub](https://github.com/ptitSeb/box64/blob/main/docs/USAGE.md).
The following settings are only the changed ones. You can leave the rest as default.

The following pictures are screenshots from a YouTube video by [Zerokimchi](https://youtu.be/EJDWZUGF9sk)

#### Box64 Unity (MonoBleedingEdge) preset
This preset represents recommended settings if your Unity game folder contains a folder named `MonoBleedingEdge`.
![winlator-box64-monobleedingedge](/assets/img/winlator-box64-monobleedingedge.png)

#### Box64 Unity (GameAssembly.dll) preset
This preset represents recommended settings if your Unity game folder contains a file named `GameAssembly.dll`.
![winlator-box64-gameassembly](/assets/img/winlator-box64-gameassembly.png)

#### Box64 Non-Unity preset
This preset represents recommended settings for all other non-unity games.
![winlator-box64-non-unity](/assets/img/winlator-box64-non-unity.png)

