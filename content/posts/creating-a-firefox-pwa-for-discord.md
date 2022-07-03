---
title: "Creating a Firefox PWA for Discord"
date: 2022-07-01T18:17:51-07:00
draft: true
---

In this tutorial, I’ll explain how to set up, install, and configure a Firefox [PWA](https://en.wikipedia.org/wiki/Progressive_web_application) (Progressive Web Application) for Discord that integrates into your system, and some background on why you’d want to do so over the regular desktop version.

## Background

If you’ve ever used Discord on Linux, you probably know that it is clunky and not nearly as polished or feature-complete as it is on, say, Windows.

Streaming has no audio, and while using [Soundux](https://soundux.rocks/) as a workaround for audio passthrough is a solution, it can be annoying to use, and doesn’t always work. For example, in my experience, Soundux fails to pick up audio from Java Minecraft (and I believe by extension all Java applications). In addition, I’ve recently had issues with audio output, with no sound being produced other than voices in VC.

Another major issue with desktop Discord is the fact it isn’t sandboxed at all, meaning it has full access to your system at any time. In addition, if you’re desktop is using X.org, Discord (and any other application, for that matter) is able to snoop on all key inputs, even if they’re from applications. Make of this what you will.

Not to mention the issues with the updater, which has been excellently covered in [this video by Brodie Robertson](https://www.youtube.com/watch?v=3OEfr7L-gUk).

There is of course a second way to use Discord on Linux, which is simply the web application. It does everything that the desktop application does, including streaming (albeit without audio), and it doesn’t ever require being updated. The only potential downside is the lack of push-to-talk, but if like me this is something you don’t use, then this isn’t an issue. In addition, since it’s just a web page not a fully-fledged application it doesn’t have the security/privacy concern of being able to access your entire system like the desktop version.

The only frustration I have with the web version is the fact that it doesn’t look like a standalone application on your desktop, it’s just a tab in your browser. Being able to remove as much of the top bar as possible would be nice. This is where PWAs come into play.

Unfortunately, support for SSB (Site Specific Browser) mode and PWAs was [dropped](https://bugzilla.mozilla.org/show_bug.cgi?id=1682593) in Firefox 86 last year.

Luckily, **Progressive Web Apps for Firefox** ([filips123/PWAsForFirefox](https://github.com/filips123/PWAsForFirefox)) exists to fill the lapse in functionality.

> This project creates a custom modified Firefox runtime to allow websites to be installed as standalone apps and provides a console tool and browser extension to install, manage and use them.

## Installation

First, install the [PWAs for Firefox extension from the Firefox add-ons store](https://addons.mozilla.org/en-US/firefox/addon/pwas-for-firefox/). Once you open it up, it will guide you through the installation process, including installing the companion application. If you’re on Arch Linux, it’s available on the AUR under `firefox-pwa` and `firefox-pwa-bin`. (`bin` for the binary precompiled version, go for this one if you don’t want to compile it yourself.)

**If you are on Ubuntu or one of its derivatives,** the snap version of Firefox that came with your system will not be able to communicate with the companion application, I’m assuming due to snaps’ containerization, at least in my testing on Xubuntu 22.04. See [this Stack Overflow answer](https://askubuntu.com/a/1404401) for a guide on switching to the standard Debian package version of Firefox.

## Create the Discord PWA

Navigate to Discord [https://discord.com](https://discord.com/channels/@me) (**not the app**, we need the root of the PWA to be the index page so all pages under the Discord domain are considered part of the PWA) and click on *Install current site* in the extension. Change the *Start URL* to [https://discord.com/app](https://discord.com/app) so the PWA opens to the app page, not the landing page. And... you’re done! You’ve created a PWA for Firefox, it’s as simple as that. You can now open it from within the extension by clicking on the external link button.

However, there’s more configuration to be done. `firefox-pwa` creates a Firefox profile for the PWAs that’s completely separate from your main profile, so we’re going to have to do some configuration there. (By default, all of your PWAs share a single *Default* profile under the *Profiles* tab of the extension, but if you want to have different PWAs have different profiles you can configure it there.)

## Configure the PWA Firefox profile

It’s up to you how want to configure the profile, but here’s what I did and recommend.

- In the :gear: *General* portion, there’s a new settings section, *Progressive Web Apps*.
  - For me I checked *Open out of scope URLs in a default browser,* which makes it so non-Discord links you visit within the PWA profile are opened in your main Firefox profile instead of within the PWA. This is also important considering any external sites you visit within the PWA aren’t otherwise going to be added to your main profile’s history.
  - In addition, I also set *Display the address bar* to *Never,* as this is not necessary and adds to the clutter of the window.
  - !!! force links into new window?
- Under :jigsaw: *Extensions & Themes*, you You might also want to [choose a theme that matches Discord’s theming](https://addons.mozilla.org/en-US/firefox/addon/discord-dark-rebrand/).
- Under :lock: *Privacy & Security* / *History*, set that history to *Use custom settings* for history and uncheck all the boxes. This will make it so the profile won’t keep any history (which is unnecessary in a PWA), but still will remember cookies so you don’t have to log in again each time you reopen it.
- If you shut down your computer with the PWA open, Firefox will think that your previous session didn’t close properly and will prompt your previous session. Since we’re only going to be the PWA profile for single websites (i.e. Discord), this is unnecessary. To disable this behavior, we’re going to need to change a couple options under the advanced configuration preferences. Within the PWA, we don’t have access to the URL bar, so to enter the preferences, press <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>I</kbd> to enter the developer tools. Enter the console and type in the following:
  
  ```JS
  document.location.href = "about:config"
  ```

  Firefox may make you type `allow pasting` beforehand.

  After this, you will enter the advanced settings page. Press *Accept the Risk and Continue*, search up `browser.sessionstore.max_tabs_undo` and `browser.sessionstore.max_windows_undo` and set both to 0.

## Launching the PWA from the command line

I have uninstalled the desktop Discord application `discord`, and I want to replace it with a shell script that launches the PWA. This will enable you to launch the PWA from the command line simply by typing `discord`, and also from an application launcher like [rofi](https://github.com/davatorium/rofi). This way, we don’t have to rely on launching Discord from the PWAsForFirefox extension.

Luckily, PWAs for Firefox provides a CLI. First, we need to find the ID of the Discord PWA:

```SH
firefoxpwa profile list
```

This will output the following. After the entry for *Discord*, the string of letters and numbers is the site ID. Your ID will be different.

```
========================= Default ==========================
Description: Default profile for all sites
ID: 00000000000000000000000000

Sites:
- Discord: https://discord.com/ (01G705MTCT1Q1HHFM7DVDFAPCY)
```

You can now launch the PWA using the following command.

```SH
firefoxpwa site launch 01G705MTCT1Q1HHFM7DVDFAPCY
```

Let’s create the launch script. Open `/usr/bin/discord` (if this exists already, uninstall the desktop Discord application) with your favorite text editor. You’ll need to use `sudo`. Add the following to the file, replacing the site ID with the one you found earlier:

```SH
#!/usr/bin/env bash
firefoxpwa site launch 01G705MTCT1Q1HHFM7DVDFAPCY
```

Finally, make it an executable:

```SH
sudo chmod +x /usr/bin/discord
```

You’re done! You’ve created a PWA for Discord that looks and behaves like a desktop application, but without a lot of the hassle involved with the actual desktop app.
