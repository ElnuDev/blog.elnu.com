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

First, install the PWAs for Firefox extension from the Firefox add-ons store. Once you open it up, it will guide you through the installation process, including installing the companion application. If you’re on Arch Linux, it’s available on the AUR under `firefox-pwa` and `firefox-pwa-bin`. (`bin` for the binary precompiled version, go for this one if you don’t want to compile it yourself.)

## Create the Discord PWA

Navigate to Discord [https://discord.com/channels/@me](https://discord.com/channels/@me) and click on *Install current site* in the extension. And... you’re done! You’ve created a PWA for Firefox, it’s as simple as that. You can now open it from within the extension by clicking on the external link button.

However, there’s more configuration to be done. `firefox-pwa` creates a Firefox profile for the PWAs that’s completely separate from your main profile, so we’re going to have to do some configuration there. (By default, all of your PWAs share a single *Default* profile under the *Profiles* tab of the extension, but if you want to have different PWAs have different profiles you can configure it there.)

## Configure the PWA Firefox profile

It’s up to you how want to configure the profile, but here’s what I did and recommend.

- In the :gear: *General* portion, there’s a new settings section, *Progressive Web Apps*.
  - For me I checked *Open out of scope URLs in a default browser,* which makes it so non-Discord links you visit within the PWA profile are opened in your main Firefox profile instead of within the PWA. This is also important considering any external sites you visit within the PWA aren’t otherwise going to be added to your main profile’s history.
  - In addition, I also set *Display the address bar* to *Never,* as this is not necessary and adds to the clutter of the window.
  - !!! force links into new window?
- Under :jigsaw: *Extensions & Themes*, you You might also want to [choose a theme that matches Discord’s theming](https://addons.mozilla.org/en-US/firefox/addon/discord-dark-rebrand/).
- Under :lock: *Privacy & Security* / *History*, set that history to *Use custom settings* for history and uncheck all the boxes. This will make it so the profile won’t keep any history (which is unnecessary in a PWA), but still will remember cookies so you don’t have to log in again each time you reopen it.
