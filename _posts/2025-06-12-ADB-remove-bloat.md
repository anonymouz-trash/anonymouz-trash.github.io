---
layout: post
title:  "Android - Remove or disable bloatware apps and even system apps!"
categories: [Android Administration]
tags: [android,adb,debug,bridge,bloat,bloatware,remove]
image:
  path: /assets/img/2025-06-12-remove-bloatware.jpg
last_modified_at: 2025-06-12 19:00:00 +0100
---

## Enabling ADB
Enabling the so called "Developer Mode" with the use of Android Debug Bridge (ADB) is very easy and very handy sometimes.
> For security reasons **don't forget to disable it** when you're done.
{: .prompt-warning}

> Depending on your device's manufacturer every menu text can look different.
{: .prompt-info}

* Go to the `settings` app of your device. Tap on `About phone`.
* Scroll down till you'll find a entry called `Build number` and tap it 6-7 times.
* If successful you'll be presented with a little notification at the bottom of the screen which says "You are a developer now!" (Or similar)

## Installation

You'll need `android-sdk-platform-tools` and there is a very well written article from `xda-Developers` which explains it:
[https://www.xda-developers.com/install-adb-windows-macos-linux/](https://www.xda-developers.com/install-adb-windows-macos-linux/)

## Usage

In my use case I use this method to get rid of bloat apps from my TV's for example. So therefor I use `Termux` on my mobile and enable `Wireless debugging` on my TV's.

Then I connect to my TV's with the following command:
```shell
adb connect <ip of device>
```

Then you have to allow this connection, as described in the article before and go to the shell of the device:

```shell
adb shell
```

## Uninstall of android apps

You'll need to identify the package names of the desired app you want to remove / disable. Just long press on the app icon and go to `App-Info`. Anywhere there you should find the package name. For example for `Netflix` it is `com.netflix.mediaclient`.

```shell
pm uninstall -k --user 0 com.netflix.mediaclient
```
> It might be a good idea to write down every package you disable. To undo changes maybe... :-)
{: .prompt-info}

## Undo changes

```shell
adb shell cmd package install-existing com.netflix.mediaclient
```

## Universal Android Debloater

This is a powerful tool written by [0x192](https://github.com/0x192) to do above described tasks automatically! You'll find it on [GitHub](https://github.com/0x192/universal-android-debloater).

![github-uad](https://github.com/0x192/universal-android-debloater/blob/main/resources/screenshots/v0.5.0.png?raw=true)
