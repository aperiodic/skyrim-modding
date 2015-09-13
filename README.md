# Skyrim Tweaking & Modding Guide

As a PC gamer, you have an unmatched level of control over the games that your computer runs.
This means that you have access to a game that's just as fun as actually playing games: tweaking, modding, and otherwise customizing your games!

This is a guide to customizing Skyrim, in my opinion the most rewarding game to tweak.
It came out near the end of the 360/PS3 console lifecycle and was targeted to run at 60 FPS on those consoles, so there's a lot of room to increase the graphical load before hitting the performance ceiling, especially with today's graphics cards.
A few months after the game's release, Bethesda took the smart but unusual step of making the tools they used to build the in-game content available for free.
In the four years since, hundreds and hundreds of nerds have made a staggering amount of third-party assets and other content for the game.

This guide is organized into three major sections:

1) [INI File Tweaks](#ini-tweaks).
   These are the easiest ones to make.
   You can reduce pop-in, increase the quality of some effects, and turn on other effects that were disabled by default.

2) [Nexus Mod Manager Mods](#mods).
   These are mods that you install using a third-party tool called Nexus Mod Manager.
   For most, you'll just need to download and apply them through the mod manager, but a few of the most useful mods (ones that overhaul the interface to make it much easier to find things) require a third party-tool called the [SKyrim Script Extender](#skse) (or [SKSE](#skse)) to be active, which has to be installed out-of-band.

3) Patches to the Executable.
   This last category is pretty much just for performance & stability.
   You should only need to do these if you're having trouble hitting 60fps but don't want to remove mods, or if you experience "Crash to Desktop" events after sustained play.
   They work by patching the Skyrim executable to fix how it performs memory re-allocation.

## Testing Technique

You'll want to be constantly testing the changes you're making, in places where they make a difference.
If the mod is for a specific place or object, then it's straightforward.
If it's more general graphics, use your favorite places.
If I want to check frame rate performance before and after a mod, I usually first load up the part of the road to the East past Whiterun where it wraps around the Throat of the World and turns South into the densely-wooded area with waterfalls and such; I'll also use a save from the plains to the West of Whiterun (near Rorikstead) where the sight distance is very large.

If you're trying to prevent Crashes To Desktop (CTDs), you'll want to make the game allocate a whopping amount of memory very quickly.
The best way to do this is to enable the console by pressing `~`, and then typing:
```
player.setav speedmult 1500
```

to make you go really fast, and then
```
TCL
```
(mnemonically 'toggle clip'), which lets you fly since you don't interact with the ground any more (no clipping) and they had the good sense to turn the gravity off for you (though you don't have inertia either).

Then just fly around the map until it crashes.
If it takes less than thirty seconds, you might get CTDs after an hour or three of play.
If you can make it for two minutes or more, you're probably fine.
Unfortunately, to make the memory allocation stuff work better, you'll need to resort to patching the Skyrim executable itself.

## INI Tweaks

There are two INI files you'll want to edit, "Skyrim.ini" and "SkyrimPrefs.ini", both located in Libraries > Documents > My Games > Skyrim.
Skyrim.ini is smaller and has more under-the-hood tweaks, SkyrimPrefs.ini is much larger and has more graphical tweaks.

*BEFORE YOU EDIT THESE FILES, MAKE BACKUP COPIES*.
Always best to err on the side of safety.
For editing these, I use notepad++, which has some built-in affordances to make editing INI files easier.

### Skyrim.ini

#### [General] Section
<a name="uGridsToLoad">

The general section contains the single tweak with the biggest impact on the graphical richness and in reducing pop-in:
```
uGridsToLoad=<n>
```
where `n` is a positive integer (the `u` stands for `unsigned`).

The Skyrim map is internally split up into a grid, and this variable's value determines how many cells around the player to load content for.
With a higher value, more content meshes are loaded, which makes the game look much better, and most importantly increases the distance at which the low-detail LOD meshes are replaced with the high-res meshes, causing a jarring pop-in affect.
It defaults to five, I have mine (on my wimpy computer with a GTX 750 Ti) set to nine, and some people run with eleven or thirteen (note that, since the value is a radius, the additional load is quadratic since it scales with the area).

There are some caveats to setting uGridsToLoad:
  * It must be an odd number, otherwise the game will not start.
  * If you don't install the [stable uGridsToLoad mod](#stable-ugridstoload) mod (which requires SKSE), then once you save a game with a high uGridsToLoad number, you can't then lower that number -- the game will crash when you try to load a save with a higher uGridsToLoad value than the currently configured one.
    With that mod installed, you can safely lower the number and still load saves with higher ones.
  * Cells that are loaded will run content and quest scripts.
    I've heard it said that, at high values, this will cause some quests to erroneously activate and bug out, but I haven't experienced this personally, so take with a grain of salt.
    If you do suspect that's happening, you should try lowering the number, and possibly using a tool that purports to "clean" your saves (this stuff gets increasingly sketchy, but supposedly the game doesn't do a good job of cleaning up all the stuff it triggered if you set the uGridsToLoad pretty high, which I'll believe -- the developers only have the resources to care about normal values).

If you change this value, you should also modify two other dependent values in the [General] Section:
  * `uInterior Cell Buffer` -- should be set to same value as `uGridsToLoad`.
  * `uExterior Cell Buffer` -- scales with area, so set to `(n+1)^2`, where `n` is the `uGridsToLoad` value.

While you're at it, you should boost up some loading buffer sizes, since I'm sure you have plenty more RAM to spare than an Xbox360:
  * `iPreloadSizeLimit=84934656` -- some sort of look-ahead content buffer size, in bytes;
    mine's set to 84 MB.
  * `fMasterFilePreLoadMB=200.0` -- seems like how much of a file to speculatively load? I have this at 200 MB.

And here's some other performance stuff I cargo-culted from other places online, without investigating very thoroughly:
  * `bPreemptivelyUnloadCells=1` -- supposed to reduce memory pressure by evicting stuff more aggresively.
  * `bUseHardDriveCache=1`
  * `bUseBackgroundFileLoader=1`
  * `bSelectivePurgeUnusedOnFastTravel=1`

#### The [Display] Section

The biggest thing here is widening up Skyrim's fairly narrow default field-of-view value:
```
fDefaultWorldFOV=85.0
```
 I like to use 85 degrees, but you should see what you like.
 Going above 90 starts to get weird, unless you have a huge or multi-monitor setup.

You can also increase how much pixel buffer space is dedicated to shadow calculation.
This is a CPU-bound activity, and if you have a recent i5 or i7 you can and should crank this up to 4096 or 8192.
I have an older i3 so I have this at the default 2048.
```
iShadowMapResolutionPrimary=2048
```

You should turn on grass shadowing the ground, something that's omitted by default but that a current graphics card should be able to handle no sweat:
```
bShadowsOnGrass=1
```

The rest of the sections I don't have anything set differently from the default, excepting

#### The [Water] Section

Let's make the water reflect reality!
```
bReflectLODObjects=1
bReflectLODLand=1
bReflectLODSky=1
bReflectLODTrees=1
```

### SkyrimPrefs.ini

This file has way more settings than Skyrim.ini.
There's a [good guide on the internets](http://www.hardocp.com/article/2011/11/23/tweaking_skyrim_image_quality/2) that hits all the important graphical settings.

## Mods

Basically, you go to [this Nexus mods website](http://www.nexusmods.com/skyrim/mods/modmanager/?), download the Mod Manager tool, and make an account on the website.
The mod manager will download mods from the website via a protocol link, and then take care of swapping in the assets while warning you when assets conflict between mods, which saves a ridiculous amount of tedium (automation woo!).
A few of the most important and useful mods require this tool called the [SKSE](#skse), short for [SKyrim Script Extender](#skse), to be installed.

### SKSE

The SKSE can be downloaded [at this "website"](http://skse.silverlock.org/) (that is adorably obviously thrown together by a developer in ten minutes).
I have mine installed standalone, not through Steam, but through Steam would probably be fine (my Skyrim *is* installed through Steam).
The SKSE installer will make a wrapper for the Skyrim binary that must be launched instead of the Skyrim binary itself (so I don't think you can't launch it through Steam, but maybe you can change the executable Steam uses).
If the wrapper is not launched then all the mods that use SKSE won't work, but this is pretty easy to notice (and I think some of them warn you) since the UI will look totally different without SkyUI).

### Game Functionality Improvements

The first mods you should install are those that just make the game more stable, make the UI easier to use, or remove bugs, without altering the game world in terms of assets or behavior or content or anything.
There are three big ones: SkyUI, Stable uGridsToLoad, and then all the "Unofficial Bug Patches".

#### SkyUI

SkyUI replaces the extremely annoying default "Giant List o' Names" inventory with a table that can be sorted by name, weight, value.
This takes a ridiculous amount of time out of inventory management and makes it much easier to figure out what to sell or consume or drop.
This is the #1 most popular Skyrim Nexus mod, because it just makes the inventory what it should have been.

[SkyUI](http://www.nexusmods.com/skyrim/mods/3863)

#### Stable uGridsToLoad

This makes it so you don't have to be totally sure you can commit in order to mess around with the [uGridsToLoad value in Skyrim.ini](#uGridsToLoad).

[Stable uGridsToLoad](http://www.nexusmods.com/skyrim/mods/41592)

#### Unofficial Bug Patches

What it says on the tin.

  * [Unofficial Skyrim Patch](http://www.nexusmods.com/skyrim/mods/19)
  * [Unofficial Dawnguard Patch](http://www.nexusmods.com/skyrim/mods/23491)
  * [Unofficial Dragonborn Patch](http://www.nexusmods.com/skyrim/mods/31083)
  * [Unofficial Hearthfire Patch](http://www.nexusmods.com/skyrim/mods/25127)

#### Better Message Box Controls

I forget what this does exactly except that sometimes I forget to load SKSE and they revert to the vanilla controls, and I'm annoyed by them.
So, this must make them better.

[Better Message Box Controls](http://www.nexusmods.com/skyrim/mods/28170)

#### Immersive HUD

Lets you configure a way to remove the HUD entirely, by having the compass disappear when you press a button.

[Immersive HUD](http://www.nexusmods.com/skyrim/mods/3222)

### Aesthetic Mods

Here's where you can just start trawling the Skyrim Nexus site sorted by popularity and install what looks good to you.
These are what I consider to be the best ones:

#### Skyrim Flora Overhaul

This modder Vurt has been making improved flora for Bethesda games for a while now.
His Skyrim Flora Overhaul adds a lot of character and variety to the in-game flora.

[Skyrim Flora Overhal](http://www.nexusmods.com/skyrim/mods/141)

#### Pure Waters

The default water shaders in Skyrim make the water look very shiny and plasticky, since they're heavy on the reflection and light on refraction.
I find this water mod to look very nice, since it's much heavier on refraction, but there are others available.

[Pure Waters](http://www.nexusmods.com/skyrim/mods/1111)

#### Climates of Tamriel

Climates of Tamriel adds much more dynamic, realistic, and in my opinion picturesque weather.
I find this to be a lot more immersive than the default highly-foggy weather, since it looks more like a real place I could be.

[Project Reality - Climates of Tamriel](http://www.nexusmods.com/skyrim/mods/17802)

If you're into this, you'll probably like the [realistic sun mod](http://www.nexusmods.com/skyrim/mods/42492), and also some [rainbows](http://www.nexusmods.com/skyrim/mods/25666)!

#### Realistic Lighting Overhaul

Changes all of the interior lighting so the dynamic lights match up with the light sources actually in the scene.
Subtle, but I found this to add a lot of realism to the interiors.
It also makes interior lighting sync up with the weather, which is another subtle but strong effect.
To make this work well with Climates of Tamriel, don't install the interior or dungeon portions of Realistic Lighting Overhaul.

[Realistic Lighting Overhaul](http://www.nexusmods.com/skyrim/mods/30450)

#### Skyrim HD

Higher-res textures.
I think they look pretty good, but I'm pretty sure there are other ones you could check out, too.
They come in up to 4K-appropriate size, if you've got one of those monitors.

[Skyrim HD](http://www.nexusmods.com/skyrim/mods/607)

#### Detailed Faces

They look a little better than the default faces, which look terrible, but they still don't look great.

[Detailed Faces](http://www.nexusmods.com/skyrim/mods/26)

#### Static Mesh Improvement Mod

This adds much detail to the objects you find in Skyrim, both their shape and texture.
The stock ones look really bad after you get used to these.

[SMIM](http://www.nexusmods.com/skyrim/mods/8655)

#### Skyrim Distance Overhaul

Increases the detail in the lower-tier Level of Detail (LOD) meshes, which reduces how apparent full-LOD mesh pop-in is (your GPU can probably handle it, but this is one of the mods it's most important to performance-test after).

[Skyrim Distance Overhaul](http://www.nexusmods.com/skyrim/mods/19446)

#### SkyFalls & SkyMills

This mod replaces the static low-LOD meshes for waterfalls and windmills with ones that are always animated.

[SkyFalls & SkyMills](www.nexusmods.com/skyrim/mods/40564)

#### Immersive Roads

These replace the stock roads with a texture that I think looks a lot nicer, and makes more sense as an endlessly tiling road (since it has large cobble stones).

[Immersive Roads](http://www.nexusmods.com/skyrim/mods/40245)

#### Quality World Map

This removes the copious amount of fog from the in-game map.
Most importantly, it draws roads on the map!
I think my character could figure out where the roads are -- they are the goddamn Dragonborn.

[Quality World Map](http://www.nexusmods.com/skyrim/mods/4929)

#### Starry Night

Some cool starry night sky textures, with overblown galaxy and nebulae you could never see with the naked eye.
Pretty cool to look at, though.

[Enhanced Night](http://www.nexusmods.com/skyrim/mods/85)

#### Bellyaches Animal and Creature Pack

What I really love about Skyrim modding is how many times nerds have looked at some small part of Skyrim, like cloaks or shields or trees, and thought "you know, I could make that way better."
This is one of my favorite examples of this -- it makes the wolves, bears, and other animals much better-looking, and adds some varities.

[Animal and Creature Pack](http://www.nexusmods.com/skyrim/mods/3621)

### Additional Items

Some mods just add more stuff you can use to the game, meaning they have some sort of gameplay impact, though it's usually pretty minor.

#### Immersive Armor & Weapons

These two mods add a buttload of styled armor and weapons that you can craft, find, or buy.
The first time you install they'll take a while to initialize and register all their new stuff with the game.
They might be more trouble than it's worth if this doesn't sound that intriguing to you.

[Immersive Armors](http://www.nexusmods.com/skyrim/mods/19733)
[Immersive Weapons](http://www.nexusmods.com/skyrim/mods/27644)

#### Cloaks of Skyrim

Someone thought "you know, Skyrim could use more sweet cloaks" and then went and made them.
Power on, you shining nerd.

[Cloaks of Skyrim](http://www.nexusmods.com/skyrim/mods/12092)

### Performance Enhancers

Some mods just try to reduce performance bottlenecks.
The only ones I've found to be effective just replace particle textures, especially the leaves in Riften, with much lower-quality versions, since they're so small and move by so quickly that you won't notice.

[Skyrim Performance Plus](http://www.nexusmods.com/skyrim/mods/6387)

### New Gameplay Mechanics

Some mods alter or add to the in-game mechanics.
I haven't used these much, but the ones I'v seen talked about the most are:

  * [Frostfall](http://www.nexusmods.com/skyrim/mods/11163) -- makes you have to pay attention to your physical condition, avoid hypothermia and whatnot.
  * [Immersive Patrols](http://www.nexusmods.com/skyrim/mods/12977) -- know how there's supposed to be a war on? Well, this adds Imperial and Stormcloak patrols that roam around and get into fights with each other.
  * [Falskaar](http://www.nexusmods.com/skyrim/mods/37994) -- Falskaar is an entire new area, with voices and everything. It's pretty well-downloaded, but who knows how good it actually is.
