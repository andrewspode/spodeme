---
permalink: /articles/reverse-engineering-a-childrens-camera/
title: Reverse Engineering a Children's Camera (and Making It Run Doom)
date: 2026-10-30
description: Dumping, dissecting and rewriting the firmware of a cheap children's camera with an undocumented SoC, and the wrong turns on the way to running Doom on it.
image: /articles/reverse-engineering-a-childrens-camera/doom-running-on-camera.jpg
tags: [Reverse Engineering, Firmware, OpenRISC, Ghidra, Doom, Embedded]
---

![Doom running on a children's camera](doom-running-on-camera.jpg)

This project is the very definition of WBIC (Why Because I Can). What started as "I wonder what firmware this runs?" ended up as the reverse engineering of an entirely undocumented SoC, building an SDK, a local emulator, a custom compiler and a fully ported version of Doom that runs on a camera with **just 891K of RAM** to play with - whilst it still works as a camera.

To be clear, this isn't Doom for the sake of Doom either. The controls are mapped well enough that you can actually play it, all the sounds work, there's chiptune music, you can save and load your game, and it runs at around 30fps.

Has this been done before? It would be easier to ask what Doom *hasn't* been ported to, but the only two examples I can find (which stupidly I looked for *after* I had finished...) are [this Russian video](https://youtu.be/fcUInR3Cl8g), where the chip (a Spreadtrum SC6531) has 4-8MB of RAM and he doesn't disclose anything about how it was done, and [this LGR video](https://www.youtube.com/watch?v=7gCZZkBATSc), where it's basically a full computer, again with plenty of RAM and a full SDK for developing the software. This was a far more significant challenge.

I chose to fully document everything on [GitHub](https://github.com/andrewspode/hx3302b-x9s-camera) - how the firmware works, a tool to dump your own over USB, the SDK, the compiler patches and the Doom port. So you can do this yourself, or take the same techniques and apply them to something similar. This article is the story of how we got there; the repo is how you can get there too.

## The Camera

![The camera, a green frog prince model](the-camera.jpg)

The box tells us nothing - no manufacturer, no address, no name, and there's nothing in the UI either. It's like it's been made to be sold on under as many brands as possible without anyone having to take responsibility for it. The only identifier is the model in the manual, X9S. The manual also claims it takes 40 megapixel photos, which I strongly suspect means the sensor is much smaller and the camera just scales the picture up.

The USB IDs brought up nothing either. This thing was practically invisible online, other than AliExpress listings from people selling them.

Then I looked at one of the photos it had taken and got a hit from the EXIF data, which led me to an article by Sprite_tm called [Kid-cam firmware modding](https://spritesmods.com/?art=kid_cam). It was a slightly different camera running quite different firmware, and the changes he wanted to make were minimal. His work gave us some good starting information (what kind of CPU to expect, for one), but it wasn't enough, and I was going to have to open the device up if I wanted to make any more progress.

## Inside

I was hesitant to open it up as my daughter loves it and I really didn't want to break it, but at this point I was too involved. Out came the soldering iron to remove the battery so I could work on it safely.

![The main board inside the camera](pcb.jpg)

Inside I found the model of the chip, which I couldn't find any other reference to anywhere, and an 8-pin flash memory chip. Reading that chip was the key to everything, and I was lucky to have the right tool on hand - an XGecu T48 programmer, which I use quite regularly when repairing motherboards. You'd be surprised how common bitrot is on 20+ year old boards.

![Close up of the SoC and the flash chip](soc-and-flash.jpg)

I have a clip that goes over the top of the chip so you don't have to desolder it from the board, but they're a pain to get on and often don't stay put. In this instance I had to hold it in place with both hands and use my little finger to hit the mouse button to read the memory. This took several attempts, and only when I had three reads that were identical did I trust that I didn't have any read errors.

Having this dump is where the real work began.

## Claude Steps In

Having recently spent quite some time on a 100% byte-matching decompile of [Indiana Jones and the Infernal Machine](https://github.com/andrewspode/infernal-machine-decomp-public) as part of helping the [OpenJones](https://github.com/smlu/OpenJones3D) project, I was very aware of how powerful AI can be at decompiling and reverse engineering - but also of its shortcomings and how to get the best out of it.

That knowledge came in handy, and it wasn't long before we had Ghidra up and running and the firmware being pulled apart into its constituents. We quickly spotted that the filename used to update this firmware was different to Sprite's - `Xly100s.bin`, rather than `hx330x_sdk.bin`. The "100s" matches the silkscreen on the board. The "Xly", I have no idea.

## Understanding the Code

Before we could do anything clever, we had to work out how the thing was put together. The camera can update itself from a file on the SD card, and the first thing we learned about that was how it writes - it starts at the beginning of the flash and writes for as long as your file is. So if you supply a 1MB file, it flashes the first 1MB and leaves the other 3MB alone. A partial update. That's a very reassuring property when you're about to start flashing a device you can't easily recover.

Additionally, although the header has a checksum, the camera never checks that it's correct. All you had to do was change the header timestamp by a second and that was in theory enough to trigger the flash. This check was supposed to happen every time you turned the camera on.

So we started tiny - the first few KB of the flash, just to confirm the mechanism worked, and that we'd be able to put the original back if we broke something. 

We were hoping we would see some sort of UI asking us to confirm the update, or that our changes to the header would mean the timestamp in the version screen would change, but as far as we could tell - nothing happened. So we spent quite some time looking for secret triggers.

During that process I discovered that if you held the shutter down on the battery screen for 5 seconds, it went to a special test mode where it said "TEST ON". We found this reference in the firmware and changed it to read "HACK ON". As this file was a lot larger, at around 770KB, we finally got a progress bar - and when we went back into the test mode, there it was:

![The camera showing HACK ON](hack-on.jpg)

The hack was indeed on - and we had been sent on a wild goose chase. Our earlier updates *were* working, they were just so small that the progress bar never had time to appear and our changes didn't make any visible difference we could spot.

Having a working flash method meant we could now start experimenting properly.

I got a lot of satisfaction out of the next one as it had always bugged me. If you go into the camera's settings, one of the menu items reads "Systerm Volume". You'd think fixing a typo would be easy - search for the text, change it, done. Except the text isn't in the firmware as text. Searching for it finds nothing, in any encoding.

![The camera's settings menu, with "Systerm Volume" highlighted](systerm-volume.jpg)

It turns out the camera has its own bitmap font with several hundred glyphs in it (Latin, Cyrillic, Thai, Chinese, Korean, Arabic...) and every string in the menu is stored as a list of glyph numbers, for each of the 18 languages it supports. We had to work out the format of those tables, write a script that drew the strings using the camera's own font so we could see what we were looking at, and then find the English string. One step closer to understanding the code, and an annoying typo was fixed.

## EXIF to the Rescue

If you haven't come across it, EXIF is a block of information stored at the very start of a photo file - things like which camera took it, when, and with what settings (on a phone it usually includes where, too). A JPEG can reserve up to 64KB for it, and this camera only uses around 730 bytes, about 1% of that, so there's a lot of spare space that nothing is looking at.

A very clever method that Sprite introduced was to use that space as a dumping ground for the camera's memory. You change the code that writes the photo's metadata so it writes a slice of the camera's own memory instead, then read it back off the SD card, inside the photo.

The basic trick only gives you one chunk of memory per photo, so we made it iterative. A counter kept in the camera's memory moved the window along each time, so every photo captured the next 4KB. Take 16 photos and you have a complete dump of the camera's internal 64KB of memory, read straight off the SD card.

That gave us a lot. It confirmed that our firmware dump was correct, as the camera's RAM matched it byte for byte. It showed that the code the firmware calls into, which we'd assumed was hidden in a ROM, was just part of the flash's own start-up code, so everything the camera runs was already in our dump. (It also exposed a mistake of ours - the "ROM" we kept trying to read came back as nothing but zeros, because we'd been working out the addresses wrongly. The memory dump is what finally let us put that right.) And by probing a megabyte at a time, we found out how much RAM the camera has - when we asked for the third megabyte, it reset. Two megabytes, then.

## Now We Test

With that, we started pulling the firmware apart properly, just trying to understand how it fitted together. We quickly found the area the games ran from, and this seemed the safest place to start modifying code. Nothing in there runs until you open the Games menu, and the update check runs as soon as the camera switches on, so however badly we broke things, we'd always have a way back.

Still writing assembly at this point, we wrote a series of small programs and benchmarks to capture information such as button presses, how much of the RAM was actually free, how things were laid out, how fast the RAM could be read and written, how fast the CPU was, and how we could interact with the display.

![The camera's 256-colour palette, drawn as a grid by one of our test programs](palette-test.jpg)

Once the results were in, we finally knew what we were working with:

- A CPU doing around 82 million simple instructions a second
- 2MB of RAM, of which just 891K was free for our own use
- 4MB of flash storage
- A 320x240 display with 256 colours, and a palette we could change

At that point I was questioning what my end goal with this device was at all. But looking at those numbers, I snorted and posed the question - do you think we could run Doom? The original Doom's system requirements asked for 4MB of RAM. We had half that, and most of it was already in use.

## Compiling Our Own Code

I haven't gone into too much technical detail here, as we have exhaustive documentation in the GitHub repo, but this camera uses an OpenRISC processor with a couple of small non-standard differences - it's little-endian, which is unusual for OpenRISC, and it doesn't use the usual branch delay slots.

To write code in something other than assembly, we needed a compiler that could speak the same language, and we couldn't find anything that did. There was an old branch of GCC that did some of what we needed, but it was abandoned and didn't do it all. So we took the mainline GCC and binutils and wrote our own patches. (PR incoming, I promise!)

In the process we also wrote a CPU emulator that follows the same rules, so that we could compile code and run it on the emulator to make sure it worked before going to the effort of flashing it to the device. That saved us a lot of back and forth - we could catch plenty of mistakes at the desk, instead of going through the whole routine of flashing and testing on the camera every time something was wrong (more on that below).

Then we replaced one of the games in the games menu with a small program that chainloads a compiled binary from the SD card. That was a big step forward, because from then on we only had to copy a new binary to the SD card to test a change, instead of reflashing the camera every time. It became the start of our very own SDK.

Debugging was the part I won't miss. There's no debugger and no serial port, so there was no real-time view of what a program was doing, and when one crashed the camera just rebooted. What we could do was have the program write logs to a file on the SD card as it went, and slowly work out where things had gone wrong. The workflow went something like this - build the program, plug the camera into the PC (where its SD card shows up as a disk), copy the program over, unplug the camera, switch it on and run it, then plug it back in and read the logs. Then do it all again. A crash meant the log just stopped dead, so we ended up writing a note after every single step and seeing how far it got. Even with AI assistance, it was a slow way of working.

## Porting Doom

We started with [doomgeneric](https://github.com/ozkl/doomgeneric), a version of Doom designed to be easy to port to all kinds of stupid devices like our camera. You provide a few functions (draw a frame, read a key, tell the time) and it does the rest.

Because we already had our own compiler and emulator, porting the code to our particular flavour of OpenRISC was fairly trivial. The real issue was how little memory we had to work with. Doom was designed for a machine with 4MB, but the camera only has 2MB, and only 891K of that is free once the firmware has taken its share. That has to hold two different things: the game's code, and its working memory - the textures, levels and sounds it loads and juggles while you play.

The working memory is the bigger problem, because Doom constantly allocates from a large pool of it while the game runs. So we went looking for more. We wrote a simple stand-in program that did everything the game would need to do - play sounds, read the keys, draw to the screen, read from the SD card - and measured which parts of the camera's memory changed while it ran. Anything the firmware never touched was a candidate for borrowing. 

Unfortunately, what looked safe often wasn't, and writing to it would reboot the camera, so there was a lot of back and forth until we squeezed out every last byte and got the total Doom could use up to around 1.4MB, albeit in lots of fragmented pieces.

That's a big improvement on 891K, but still a long way short of the 4MB Doom was written for, on a camera whose entire RAM is 2MB. More trickery was going to be needed.

![The famous 640K RAM quote attributed to Bill Gates](640k-ram.jpg)
*Bill Gates almost certainly never actually said this, but it felt relevant.*

The fragmentation was a problem in itself. Doom constantly allocates and frees small things, and over time that chops memory into pieces too small to use, until the big allocations it needs can't find a contiguous chunk. So we gave the small allocations their own reserved area, to stop them getting in the way of the large ones.

Even then there still wasn't room for all of the game's data at once. All of it (the graphics, sounds and levels) lives on the SD card inside the WAD file, and the game reads in what it needs and throws it out again when it doesn't. The trick was deciding what to keep and what to eject so that performance didn't suffer - level data, for example, is cleared out when you leave a level, and the sound effects can be thrown away and read back in whenever memory gets tight.

The game's code was a separate problem, and it was competing with all of that for the same RAM. Normally you'd load the whole program into RAM, but we didn't have anywhere near enough room. The code can run straight from the flash chip instead, but when we benchmarked that, we found it's brutally slow - code that doesn't fit in the CPU's tiny cache ran about 18 times slower from flash than from RAM. It doesn't have to be one or the other, though. Some of the code could live in RAM and the rest in flash, so the question became which parts.

To find out, we profiled Doom on our emulator, recording which parts of the code actually ran as it played. It turned out that 99% of the time was spent in about 29KB of code out of 260KB; all the rest was start-up code, menus, the automap, intermission screens and so on. So we put that small hot core in RAM, and everything that rarely runs stays in flash, where being slow doesn't matter. That freed up hundreds of KB of RAM for the working memory, and Doom ran at the same speed as the all-in-RAM version.

We also sped things up by bypassing the firmware's own event-driven key system and reading the keys directly. The firmware's main loop turned out to cost around 38 milliseconds of every frame all by itself, so we wrote our own game loop instead, which roughly doubled the frame rate. Reading the keys directly also let us use the power button as a game button instead of having it simply turn the camera off.

Once we had the game running the way we wanted, we moved on to sound and music. The camera can only play a single stream of audio, whereas Doom expects to play many sounds at once, so we wrote a software mixer that mixes all of Doom's sound effects into that one stream.

Music was harder. Doom's music isn't recorded audio, it's a compact score in a format called MUS - a lot like MIDI, a list of "play this note on this instrument at this volume". On a PC, Doom plays it with an FM synthesiser chip, which the camera doesn't have. We could have emulated that chip, but it was far too heavy for the CPU. We also considered pre-rendering the music as MP3s and using the camera's built-in decoder, but that just didn't feel authentic.

What we did instead was write a very small synthesiser of our own. It reads the score and plays it with 16 voices, each one a simple waveform (a square, triangle or sawtooth wave, depending on the kind of instrument) with a basic volume envelope, and uses noise for the drums. It's mixed into the same stream as the sound effects, and costs about 2% of the CPU. The result is Doom's classic music played like an old games console - very chiptune. Personally, I love it.

Before putting any of it on the camera, we compiled the synthesiser to run on the PC and rendered the first music files there, so I could vet them. We had to decide whether they were good enough, and I loved them straight away. Being able to test on the PC sure came in handy. Here are the first 30 seconds of "At Doom's Gate" (E1M1) and "Dark Halls" (E1M3), both composed by Bobby Prince, from that test - rendered on the PC using the same code that runs on the camera, rather than recorded from the camera itself.

<p>E1M1, "At Doom's Gate":</p>
<audio controls preload="none" style="width: 100%;" src="e1m1-camera-synth.wav">Your browser doesn't support audio playback.</audio>

<p>E1M3, "Dark Halls":</p>
<audio controls preload="none" style="width: 100%;" src="e1m3-camera-synth.wav">Your browser doesn't support audio playback.</audio>

## How Do You Control Doom on a Camera?

Thankfully we have four arrow keys, which solved the biggest problem, but we had to make some compromises - or perhaps clever decisions - for the rest of the buttons.

The shutter fires. Kind of obvious, and holding it down keeps firing, which we could only do once we were reading the keys directly.

To switch weapons we use the power button - a single click cycles through the weapons until you find the one you want. A long press works as escape and takes you to the menu. Once you're in the menu, a short tap is all you need to go back.

Because we'd taken over the power button, it no longer turns the camera off. So to switch off, you use Doom's own "quit game" menu option, and confirm with the shutter (which acts as enter).

Load and save usually ask you to type a name for the saved game, so we made them name themselves after the slot instead (Save 1, Save 2 and so on).

Finally, there was no spare button for "use", so we detect when you walk up against something and keep pushing, and it activates it. Step back and walk into it again and it activates a second time, which is ideal for turning switches on and off. It works well, and has the nice side effect that you can just charge at doors and have them open without needing a button.

Here's a video of it working:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%;">
<iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" src="https://www.youtube.com/embed/9zbToc-B2Ns" title="Doom running on a children's camera" frameborder="0" allowfullscreen></iframe>
</div>

There's a bit of struggle when the menu is open during heavy gaming. I haven't tried to improve it, as it doesn't affect gameplay and it's an edge case not worth worrying about.

This took a lot longer than I expected, but I'm super happy with the end result - it feels flawless to play. From the first flash to Doom running took just two days, including some very late night debugging.

## Reading the Firmware Over USB

Once Doom was working, I wanted other people with similar cameras to be able to use our work too. The obvious answer would have been to share my firmware dump, but that felt questionably legal. Even if I'd reverse engineered the whole thing back to C, there would still be assets in there that aren't mine to share. So instead I wanted to give people a way of dumping their own, without needing to open the device up. Unfortunately there was no direct interface for it.

What we did find was a SCSI command on the USB device that let the computer ask the camera to run code at any address. It's the kind of thing that could be used fairly universally for getting at the device, but any attempt to get Claude to do anything with it was shut down for security reasons, which halted that line of work.

Thankfully my good friend [Jack](https://jpb.dev) had a Kimi subscription and let me borrow it for a few hours. Kimi had no such qualms. It ripped through the firmware, wrote a series of tests to see what was going on and ultimately came up with a very clever technique - writing a small stub of code into the camera's RAM that can read the flash memory and copy it out in small chunks, then running it over and over until you've read the lot. The camera's own CPU becomes a proxy for reading its own flash.

Having the original dump from the programmer meant we could check it worked, and it matched. Not much use to me now, but hopefully useful to someone else! The big caveat is that it uses specific addresses in this firmware, so it may well only work on this exact version - I'd love to hear if anyone tries it on another.

## What I'd Do Differently / What's Next

What would I do differently? I'd put the USB cable down and say, Jesus Christ, Spode, find something better to do with your time.

However, we now have a lot of documentation, and an SDK of sorts, for something that previously didn't even exist on the internet. I've put the original firmware back before handing the camera back to my daughter, but now I can start on phase 2, simplifying the camera's UI, and updating the games with a few of the ones I have in [Spode's Playground](/software/playground/). Once she's bored, I'll reinstall Doom and it can sit on my shelf as a trophy.

If you have one of these cameras (or one that looks like it), the repo has everything - a tool to dump your own firmware over USB without opening it up, and a script that turns your dump into a Doom-enabled update. I'd love to hear how you get on.
