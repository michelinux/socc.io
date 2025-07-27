---
layout: post
title: Backporting TrackWeight app on a vintage Mac with Gemini-Cli
date: 2025-07-27 02:45 +0200
published: true
---

The latest funny thing I saw around, is the [TrackWeight app on MacOS](https://github.com/KrishKrosh/TrackWeight) which allows you to use the Trackpad of your Mac to weight stuff. If you have a recent Mac and a Trackpad, just install and play:

```bash
brew install --cask krishkrosh/apps/trackweight
```

Else, should your MacBook Pro 2017 still be going strong, here is a post for you.

---

The Trackpad (both the one on MacBooks and the external Magic Trackpad) it actually doesn't click. It pretends to do it. It's called Force Touch and it uses two neat tricks:

1. it simulates the click with some haptic feedbacks. You feel it's clicking but the trackpad is not moving (it's moving a bit);
2. it measures how strong you are pushing your fingers on the pad (that is not technically _pressure_ because it's not a _Force_  over an _Area_).

The second trick is what is used in TrackWeight app. You keep one finger on the pad to trick it into thinking you are actually using it, and then thanks to the OpenMultitouchSupport lib, the app can measure how much weight you have put on it. It works and it's precise enough for family cooking, although I wouldn't advise keeping a multi-thousands euro machine near liquids, flowers, etc.

While the MacBooks Pro have the Force Touch since 2015, the WeightTrack app does not work on older machines: it uses Swift 6, and that's not the only problem. I wanted to backport it and run it on my personal laptop which i stuck on MacOS 13 Ventura. I thought: "How difficult would it be?". It wasn't that difficult.

Two big points before we continue:

- I know nothing about coding in Swift ([despite a talk](https://youtu.be/nA0Yr-LZENA?t=1628))
- I have access to Gemini-Cli (and ChatGPT for the last bit - read on).

### The naïve way

I first tried to just checkout the code of TrackWeight and ask to just make it work. Gemini-Cli went completely off track at the beginning, but after I gave it a little more context it understood the problem and correctly targeted Swift 5.9. But then it went way off course when it realised the problem also came from the OpenMultitouchSupport dependency, whose precompiled version was not compatible (not sure because of MacOS version or because of CPU architecture).

Result: `git checkout -- .` and start again.

### A bit more Human Guidance

I decided to _attack_ the backport of TrackWeight and OpenMultitouchSupport independently.

#### Backporting `OpenMultitouchSupport`

I checked out the OpenMultitouchSupport and asked Gemini-Cli to make it compile with Swift 5.9. It consumed all the tokens I could use for the day on the _Pro_ version of the model but it worked. Until it tried depsperately to find an executable and started doing wild changes to the code in order to obtain one.

Luckly I did commit the changes as soon as the built passed and stopped its trip.

In the end it only made three changes:
1. Modified `Package.swift` to Swift 5.9;
2. Backpedalled some API from Swift 6 to Swift 5.9;
3. Disable the signing (I do not have an Apple Developer ID, and I don't think 99 USD are worth a glorified kitchen scale).

##### Backporting `TrackWeight`

TrackWeight required more work, and more guidance.

It started well but had to stop Gemini-Cli when it started to look for the OpenMultitouchSupport library. I told it that it could find it in `../OpenMultitouchSupport` folder, but it was adamantly refusing to leave the current folder. I manually run a `ln -s ../OpenMultitouchSupport` and asked it to look for the lib in the current folder. It was very happy about it.

After a few back and forth it got to fully compile. But it didn't run because it was not compatible with MacOS13.

Tried again and after changing target to MacOS 13 and adapting some API that were not available yet in that version I got another `.app`. Which again didn't work, this time with a big crash because _"mapped file has no Team ID and is not a platform binary (signed with custom identity or adhoc?)". At this point, once again, Gemini-Cli started its usual wild editing to what I understand was an attempt to bypass the signature of the binary.

I stopped it, and asked ChatGPT, which as the senior LLM that it is, quickly said to just:

```
codesign --force --deep --sign - ~/Library/Developer/Xcode/DerivedData/TrackWeight-*/Build/Products/Release/TrackWeight.app
```

which signs the app with an ad-hoc identity (whatever that means).

Changes in the end were similar to the OpenMultitouchSupport:

1. Target Swift 5.9 **and** MacOS13;
2. Adapt API to the older version of MacOS (I don't think there was any change due to Swift);
3. Use the local OpenMultitouchSupport lib instead of the remote one;
4. Manual sign the app (not really a change, it's a manual operation)

### Considerations

![It's almost correct](/assets/img/2025-07-27-barilla.jpg){: .right .w-25}

It was relatively easy to do, as changes were not that big. Without Gemini-Cli (or any other AI tool) it would have taken me far longer and I would have not even approached it and more savily I would have spent a few hours reading a good book on the beach.

I am not going to publish any compiled version, nor I am going to maintain it. You can find the changes committed in:

* [https://github.com/michelinux/OpenMultitouchSupport/tree/macos13](https://github.com/michelinux/OpenMultitouchSupport/tree/macos13)
* [https://github.com/michelinux/TrackWeight/tree/macos13](https://github.com/michelinux/TrackWeight/tree/macos13)

### Something I don't fully understand 🤷‍♂️

The `project.pbxproj` contains all the dependencies and their UUID. What happens with the local version of a library? Do I get a new UUID each time I change something?

### Small rants section

Ran number one. The pressure support was present also on iPhones and on Apple Watch, and it was called 3D Touch. My Apple Watch 5 did have it and I used it a lot to watch live photos. Following models did not and with an update Apple decided to disable it from the Watch 5. It was to harmonise the experience across the line, but I paid good money (well, it was a gift) also for that feature. A bit like saying that since those who had a recent model didn't have 3d Touch, why should a poor guy with an older model still enjoy it? The result is that I don't watch photos anymore on my watch.

Another rant is about Swift 6. Once can easily install Swift 6 on an old computer running Linux (I tried, it works), but for some reason if your MacOS is 2 years old you are already completely left behind.
