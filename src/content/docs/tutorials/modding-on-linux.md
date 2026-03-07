---
title: Modding on Linux (a Guide for Newbies)
description: A tutorial for modding The Sims 4 on Linux
lastUpdated: true
sidebar:
  order: 50
  badge:
    text: UPD
    variant: note
---

<sup><sub>A tutorial by OneMoreKayaker</sup></sub>

A lot of folks are switching to - or have recently switched to - Linux, including myself, and there's been a lot of interest in learning how to mod the Sims 4 in a Linux environment.

This tutorial is aimed at getting the basic modding tools for Sims 4 set up to work on Linux, and is written to assume a minimal level of familiarity with the OS. As I use Linux Mint 22 myself, it is written with that distro in mind.

The tools we are focusing on are Sims 4 Studio, XML Extractor and Finder, and Blender. You do not need a DocFetcher alternative on Linux Mint Cinnamon, the basic file manager has the ability to search all of the files in a folder and set of folders and also search the file contents.

## **Step 1: Getting Sims 4 Set Up**

As of early 2026, the best way to get Sims 4 set up on Linux is to use Steam and its Proton compatibility layer. Install Steam, using your method of choice. Next, connect your EA account to your Steam account, if you haven't already done so. This step is crucial if you have any DLC from EA.

Start the download process for installing Sims 4 from Steam. Steam will install the Sims 4 base game and a lightweight version of the EA app, as well as any DLC you purchased from Steam. Provided you linked your EA account, the EA app will then download the DLC you purchased directly from EA.

In your Steam library, right-click on the Sims 4 game title and go to Properties -> Compatibility -> Proton Experimental. As of March 2026, this is what you need to run Sims 4 on Linux Mint Cinnamon.

Depending on how much DLC you own, the download process may take approximately fifty bajillion years. So we'll proceed to the next step while that runs.

## **Step 2: Installing WINE and WineTricks**

This tutorial uses WINE and WineTricks to enable Sims 4 modding tools (designed for Windows) to work on Linux. One thing to know about Sims 4 Studio is that it works best with WINE version 10.0. This version is not currently available in the Linux Mint repos (or repositories), but can be downloaded directly from Wine HQ (the organization that makes WINE) with a little bit of finagling.

Note that these instructions are for Linux Mint version 22. They will not work as-is for other OSes! You will have to look up some things and edit as needed.

First, please open the Linux terminal. Then, in the terminal, paste <u>each individual line</u> using "CTRL+SHIFT+V" to paste:

<details><summary><b><i>Did you already install a different version of WINE? Please remove it first!</i></b></summary>

If you have the wrong version of WINE installed, either version 11 or an earlier version, you'll want to completely remove it before trying to install WINE 10.0.

These instructions will take you most of the way there. Please do this before trying to install WINE 11.

```
sudo apt remove --purge wine
sudo apt remove --purge wine winehq-* wine-stable wine-devel wine-staging
sudo apt autoremove
```

You may also need to manually search for and remove some extra things. You can use the commands "grep" and "rm" to find and delete these extra files.

```
grep -R "winehq" /etc/apt/sources.list /etc/apt/sources.list.d
grep -R "wine" /etc/apt/sources.list /etc/apt/sources.list.d
```

If you find anything that looks suspicious, remove it like so:

```
sudo rm /etc/apt/sources.list.d/winehq.list
sudo rm /etc/apt/sources.list.d/winehq-jammy.sources
sudo rm /etc/apt/sources.list.d/winehq-noble.sources
```

Just paste in the full filepath that you discovered with grep.

Once everything is gone, do sudo apt update to finish.

</details>

```
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wget gnupg2 software-properties-common
sudo mkdir -pm755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources
sudo apt update

```

After completing these steps, you should check to make sure that WINE version 10.0 is available for you to download. Enter the following command into the terminal:

```
apt list -a winehq-stable
```

The output should list version 10.0. If it does, proceed to the next step. If not, you may need to either use version 9.0 (it should be available in the Linux Mint repository) or attempt to compile version 10.0 from source.

Please paste the following five lines of code <u>all together</u> into the terminal. The backslashes tell the terminal that this is a single command.

```
sudo apt install --install-recommends \
winehq-stable=10.0.0.0~noble-1 \
wine-stable=10.0.0.0~noble-1 \
wine-stable-amd64=10.0.0.0~noble-1 \
wine-stable-i386=10.0.0.0~noble-1
```

Verify that the right version of WINE is installed:

```
wine --version
```

To lock in version 10.0 and prevent WINE from automatically updating to version 11.0, please use this command. The showhold command will tell you if it worked. Paste them separately.

```
sudo apt-mark hold winehq-stable wine-stable wine-stable-amd64 wine-stable-i386
apt-mark showhold
```

In the Update Manager (the shield icon on your panel/task bar), hit the "Refresh" button to make sure the hold is recognized and prevent accidentally downloading a newer version of WINE.

If you do need to update to a more recent version of WINE (for example, if the bugs with 11 are fixed), you can use this command to remove the hold:

```
sudo apt-mark unhold winehq-stable wine-stable wine-stable-amd64 wine-stable-i386
```

This completes the WINE installation. If, during any of this, you are asked to install Mono, please install Mono. You might also be asked the first time WINE runs. Please install it then.

After you've installed WINE, you can, from the Terminal, install WineTricks. This time it's just:

```
sudo apt install winetricks
```

Once WineTricks is installed, launch it, either by typing "winetricks" into the Terminal or by going to the Menu and clicking on its icon. When it launches it'll give you a brief warning about "using a 64-bit wineprefix" but you can ignore that - that only applies if you're trying to install a 32 bit program into the default prefix, which we're not going to do (and please don't do that, it's a bad idea).

Click on "Select the Default Wineprefix" and hit "OK". From the "What would you like to do to this wineprefix?" screen, choose "Install a Windows DLL or component" and hit "OK."

Sims 4 Studio needs the Windows .NET Runtime 6.0 and .NET Desktop Runtime 6.0 to work. So we're going to navigate to find "dotnet6" and "dotnetdesktop6" and select them, then press "OK" to install them.

Once they're installed, we need to install fonts. The fonts are a critical part of using WINE, as missing fonts will cause Windows programs (including Sims 4 Studio) to crash on startup.

From the "What would you like to do to this wineprefix?" screen, select "Install a font." Select "allfonts," and then "OK." Linux will then proceed to install... not *all fonts,* but certainly most of them.

You may need to click through several menus about a running process - just hit "okay" so the fonts can continue to install.

With WINE and WineTricks set up, you are now ready to install your Sims 4 modding tools.

## **Step 3: Setting Up Sims 4 Studio for Basic Use and Tuning Modding**

Grab the latest version of Sims 4 Studio from the Sims 4 Studio website, and download the installer (yes, the *installer*, not the zip file). Then, run the installer as you would on a Windows machine. If you need to select a program to open it, please select the "Wine Windows Program Launcher." It should install in the way you'd expect.

<details><summary><b><i>Have a high display resolution monitor? Use a wineprefix!</i></b></summary>

If you have a high display rate monitor, you might have noticed that some programs are teeny-tiny by default. Programs launched from WINE are not scaled by your main operating system: you need to set the scaling manually by using WineTricks to edit the wineconfig and set screen resolution to the proper DPI (multiply the default DPI of 96 to whatever you'd like, but use a whole number. 200% scaling = 192 DPI, which is 2 x 96.)

Changing the default WINE scaling *can* cause problems with some programs, including XML Finder and Sims 4 Tray Importer. They might work very slowly, or might not work at all. This is not ideal.

The way *around* this problem is to create a separate *wineprefix* using WineTricks, and only apply the scaling to that. That way Sims 4 Studio and other programs are at a comfortable size, and the programs that can't be scaled work normally.

You can do this by launching WineTricks, selecting "create new wineprefix", and editing wineconfig to the scaling you want. Make sure to install your fonts and the dotnet requirements in this new wineprefix!

Once that is done, use WineTricks to select your new wineprefix (I named my "ScaledPrograms"), then select "run an arbitrary executable" and select the Sims 4 Studio installer to install it directly into the prefix.

If you do not have a high display resolution monitor you should be fine using the default wineprefix, unless dotnet 6.0 or dotnet 6.0 runtime causes issues with your other programs. Then, just create and use a new prefix.

</details>

Once Studio is installed, open it to make sure it works. If it crashes, you might have missed the .NET or font installation, or installed them to something other than the default wineprefix, so please revisit those steps.

With Studio open, you will be directed towards the Settings menu. Please *deselect* "Disable Hardware Rendering!" having this selected will cause Studio to crash when browsing game content or editing CC.

Now you need to add your filepath to the game installation. Provided you installed via Steam, the path should look something like:

```
/home/yourusername/local/share/Steam/steamapps/common/The Sims 4
```

To make it so that Sims 4 Studio will automatically open package files when you click on them, you will need to take an additional step. During the Sims 4 Studio installation process, WINE should have automatically created a shortcut in your Menu. Navigate to that shortcut now.

Right-click on the Sims 4 Studio shortcut in the Menu, and select "Properties." Copy the text inside the command line. Then create a .package file using Studio (or download one, it all works) and right-click on it. Select "Open with Other Application" and paste the command you copied into the line on the bottom. Then select "OK."

Try opening the package by clicking on it. Studio should open it easily.

If you want to pin Studio to your taskbar (called the "panel" in Mint), you can do that by right-clicking on the shortcut in the main Menu. Once added to the panel, you can right-click on it, select "Applet preferences" and "Configure" to choose if you want a type of mouse click to launch a separate instance. I have mine set to launch a new instance when I click on it using the middle mouse button.

We will touch on Blender and how to set that up for use with Studio on Linux later in this tutorial.

## **Step 4: XML Extractor and Finder**

XML Extractor and Finder are highly useful for any type of tuning modding, as they identify the relationships between different tuning files. This is critical when working on more complex gameplay mods, which are sets of tuning files working together as a system.

These programs do not have an installer, so once you download them you can just plunk them down wherever. I have a folder called "Applications" that I like to use for random things like this.

Something to note: Linux doesn't like spaces in names. This can create issues with filepathing and some default functions. So if you want to create a working shortcut for these programs, or to put them into the Menu or pin them to the panel, I'd recommend renaming them to "XMLFinder.exe" and "XMLExtractor.exe" with no spaces.

XML Extractor, to extract the game files, will need the same filepath to your Steam game installation that you provided to Studio earlier. So paste that in. If your DLC is still installing, don't start Extractor just yet. Once the DLC is all installed and the Sims 4 game is closed, you can extract to a folder of your choice.

If you'd like to create shortcuts to get to these programs, provided you removed the spaces from their names, and once you've opened them from Applications or wherever you put them, right-click the icon in the panel and select "Create Shortcut" to add them to the Menu. The Menu shortcut should, in its Properties section, have a command to open them directly in WINE - if not, just type in "wine home/yourusername/filepath/filename.exe". You will need to do some backslash nonsense if you left spaces in.

You can right-click the shortcut in the Menu to add them to your panel.

## **Step 5: Creating a Proper Work Environment**

Finding the filepath to your mods folder is a bit more of a pain than getting the filepath to the actual Steam game. It should look something like this:

```
/home/yourusername/.steam/steam/steamapps/compatdata/1222670/pfx/drive_c/users/steamuser/My Documents/Electronic Arts/The Sims 4
```

Linux has something called a "symbolic link" that is a kind of super shortcut that you can use to get here faster. Click on the Sims 4 folder where your mods and saves live (in the pseudo "Documents" as per the previous codeblock) and hit "CTRL+M" to create a symbolic link. The symbolic link can be dragged anywhere you would like it to go. You can also create symbolic links from the Terminal.

Because Sims 4 Studio is running on a compatibility layer in WINE it can't actually see all of your folders and files in Linux. Symbolic links have a strange superpower in that they can actually be used in filepaths, allowing Studio to access whatever you want it to be able to get to.

I've added symbolic links to my Desktop and Documents folder, targeting my Downloads folder, Sims 4 docs folder, and my modding projects, to make it easier for Studio to import from and export to.

## **Step 6: Blender and GIMP**

Setting up Blender is a more complicated process.

You need to download the Windows Portable version from the Blender site (the one that comes in a zip file) and *not* the Linux version (Studio doesn't know what to do with it) or the installer (that is a .msi and won't actually install.) I put mine in my "Applications" folder alongside XML Extractor and Finder and my other random applications.

Once the WinPort version of Blender has been put in an appropriate home, you'll need to manually install several files into the 3.6 folder (I'd recommend using v3.6 at least for exporting files as that tends to have fewer potential bugs, especially for CAS creators). The files and instructions can be found here on the official Sims 4 Studio site: https://sims4studio.com/thread/38676/blender-install-add-ons-solved

Once Blender is set up, give Studio the filepath to the installation as you normally would.

To open your .blend files when clicking on them, you will want to install the Linux native version of Blender. You can do that from the software manager, or directly from Blender if you want a newer version. Note that the reverse might not be true! Also, use a version that Studio supports (currently up to v5.0).

Many CC users will also use PhotoShop for a number of steps and processes. PhotoShop does not work on Linux, and generally people who want to use Linux will migrate to GIMP or Krita instead, with GIMP the most common destination.

However, PhotoShop users may experience some challenges adjusting to the GIMP UX and getting it to do what they want it to do. There is a project called "PhotoGIMP" that allows users to mod GIMP so its closer to PhotoShop. This project can be found here: https://photogimp.com/

## **Additional Tips and Tricks**

If you have a high display resolution monitor, you may have encountered teeny-tiny text when launching a program through Wine. To fix this, you'll need to go into your wineprefix using WineTricks, run wineconfig and then go to the "Graphics" tab and set your DPI scaling to 192 (for 200% scaling). This works great for Sims 4 Studio and the WinPort version of Blender, as well as most programs you'll launch via Wine.

*However,* some programs built using older methods experience large performance hits once you start using DPI scaling. XML Finder + Extractor, Tray Importer, and Denton47's Premade Household Tool are among them. I'd recommend creating a new wineprefix (WineTricks -> Create new wineprefix -> 64bit architecture) for those and keeping DPI scaling at 96 to ensure an acceptable level of performance. Remember to install allfonts!

You can instruct the programs to launch in the new wineprefix by adding the additional 'env WINEPREFIX="path to prefix"' to their launchers:

```
env WINEPREFIX="/home/yourusername/.local/share/wineprefixes/YourNewWinePrefix" wine /home/yourusername/Applications/NameOfProgram.exe
```

Please replace with the actual name of the wineprefix and the actual path to your application.

I hope this tutorial was helpful to you, or at least informative.

---

Originally posted on the [Creator Musings Discord server](https://discord.com/invite/qxz5Kn5).