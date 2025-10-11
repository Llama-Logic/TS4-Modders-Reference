---
title: Modding on Linux (a Guide for Newbies)
description: A tutorial for modding The Sims 4 on Linux
sidebar:
  order: 50
---

<sup><sub>A tutorial by OneMoreKayaker</sup></sub>

A lot of folks are switching to - or have recently switched to - Linux, including myself, and there's been a lot of interest in learning how to mod the Sims 4 in a Linux environment.

This tutorial is aimed at getting the basic modding tools for Sims 4 set up to work on Linux, and is written to assume a minimal level of familiarity with the OS. As I use Linux Mint 22 myself, it is written with that distro in mind.

The tools we are focusing on are Sims 4 Studio, XML Extractor and Finder, and Blender. You do not need a DocFetcher alternative on Linux Mint Cinnamon, the basic file manager has the ability to search all of the files in a folder and set of folders and also search the file contents.

## **Step 1: Getting Sims 4 Set Up**

As of 2025, the best way to get Sims 4 set up on Linux is to use Steam and its Proton compatibility layer. Install Steam, using your method of choice. Next, connect your EA account to your Steam account, if you haven't already done so. This step is crucial if you have any DLC from EA.

Start the download process for installing Sims 4 from Steam. Steam will install the Sims 4 base game and a lightweight version of the EA app, as well as any DLC you purchased from Steam. Provided you linked your EA account, the EA app will then download the DLC you purchased directly from EA.

In your Steam library, right-click on the Sims 4 game title and go to Properties -> Compatibility -> Proton Experimental. As of October 2025, this is what you need to run Sims 4 on Linux Mint Cinnamon.

Depending on how much DLC you own, the download process may take approximately fifty bajillion years. So we'll proceed to the next step while that runs.

## **Step 2: Installing WINE and WineTricks**

This tutorial uses WINE and WineTricks to enable Sims 4 modding tools (designed for Windows) to work on Linux. There are other methods that you can also use, such as Bottles, but I'm going to focus on WINE as I believe this works a bit better for this use case.

One thing to know about Linux Mint is that its version of WINE (the copy of WINE stored in the Linux Mint app library, aka the repository or "repo") is a bit old and outdated. This can cause issues with some Windows programs. So, you will want to install a newer version of WINE - and we will use the version intended for Ubuntu, as Mint is based on Ubuntu.

Please open the Linux terminal. Then, in the terminal, paste each individual line using "CTRL+SHIFT+V" to paste:

```
sudo dpkg --add-architecture i386
sudo mkdir -pm755 /etc/apt/keyrings
wget -O - https://dl.winehq.org/wine-builds/winehq.key | sudo gpg --dearmor -o /etc/apt/keyrings/winehq-archive.key -
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources
sudo apt update
sudo apt install --install-recommends winehq-stable
```
<details>
<summary>What is this doing, exactly?</summary>

Most of the time, when you want to install a program in Linux, you can use a typical "sudo apt install whatever" command from the Terminal, or find your program in Software Manager, or grab a .deb file or an AppImage from an official website if its not in the repo from your distro (the repositories the Linux Mint team - or whichever distro you use - maintains are the ones you get from "sudo apt install whatever" or the Software Manager).

Some companies and sources might also give you instructions to install directly from their own repo, or repository, using the Terminal. The advantage of doing this is that you generally get faster updates than waiting for the version from your distro. The disadvantage is that sometimes you run into compatibility issues and bugs, so you should approach this on a case-by-case basis. For Linux Mint it is good to get WINE directly from WineHQ.

A step-by-step breakdown of the above Terminal instructions:

```
sudo dpkg --add-architecture i386
```

This is instructing your computer to add support for 32 bit architecture, if you don't already have it. Because WINE needs 32 bit architecture for some of the programs it supports, you want to do this or else havoc ensues. This command works for Debian-based systems specifically - Debian is basis for Ubuntu, which is the basis for Mint, and they use the same stuff for this process.

```
sudo mkdir -pm755 /etc/apt/keyrings
```

Linux uses something called a "keyring" to keep track of sensitive data and will provide access to this data to the programs that need it - on a restricted basis. This command is telling your computer to make the keyring directory if it doesn't already exist and the "pm755" command is giving your Terminal the ability to temporarily read, write, and execute in here.

```
wget -O - https://dl.winehq.org/wine-builds/winehq.key | sudo gpg --dearmor -o /etc/apt/keyrings/winehq-archive.key -
```

This command is getting the secret key from WineHQ that proves its identity and is adding it to your keyring, so your computer will know WineHQ is the right source for WINE updates.

```
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources
```

This command is the one that actually adds WineHQ to your software sources list and gives your computer the location of the WINE repository. Note that this command directs your computer to the respository intended for *Ubuntu,* which also works for Mint.

```
sudo apt update
sudo apt install --install-recommends winehq-stable
```

These two lines (and they are *two* lines, please paste them in separately) will - for lack of a better term - update your Update Manager and then actually install WINE. You should do "sudo apt update" before installing things generally, it's a good idea and helps you avoid weird bugs and dependencies issues.

The version of WINE we're installing is the WineHQ-Stable one, which is stable and generally the least bugged. There's also a WineHQ-Staging version, which is the the most recent testing version, and a third version that has some corporate use that we don't care about.

Staging might let you avoid certain bugs, but it will also introduce other new and exciting ones, so we're just going to use Stable instead.

</details>

This will install the most up-to-date version of WINE, directly from WineHQ. Note that this is compatible with *Mint* and *Ubuntu*, if you have a different distro you will want to use a different set of instructions that are intended specifically for your distro. Do not use this one!

If, during the WINE installation, you are asked to install Mono, please install Mono.

After you've installed WINE, you can, from the Terminal, install WineTricks. This time it's just:

```
sudo apt install winetricks
```

You can also use the Software Manager, but I find the Terminal is a bit faster.

Once WineTricks is installed, launch it, either by typing "winetricks" into the Terminal or by going to the Menu and clicking on its icon. When it launches it'll give you a brief warning about "32 bit prefixes" but you can ignore that - that only applies if you're trying to install a 32 bit program into the default prefix, which we're not going to do (and please don't do that, it's a bad idea).

Click on "Select the Default Wineprefix" and hit "OK". From the "What would you like to do to this wineprefix?" screen, choose "Install a Windows DLL or component" and hit "OK."

Sims 4 Studio needs the Windows .NET Runtime 6.0 and .NET Desktop Runtime 6.0 to work. So we're going to navigate to find "dotnet6" and "dotnetdesktop6" and select them, then press "OK" to install them.

Once they're installed, we need to install fonts. The fonts are a critical part of using WINE, as missing fonts will cause Windows programs (including Sims 4 Studio) to crash on startup.

From the "What would you like to do to this wineprefix?" screen, select "Install a font." Select "allfonts," and then "OK." Linux will then proceed to install... not *all fonts,* but certainly most of them.

With WINE and WineTricks set up, you are now ready to install your Sims 4 modding tools.

## **Step 3: Setting Up Sims 4 Studio for Basic Use and Tuning Modding**

Grab the latest version of Sims 4 Studio from the Sims 4 Studio website, and download the installer (yes, the *installer*, not the zip file). Then, run the installer as you would on a Windows machine. If you need to select a program to open it, please select the "Wine Windows Program Launcher." It should install in the way you'd expect.

Once Studio is installed, open it make sure it works. If it crashes, you might have missed the .NET or font installation, or installed them to something other than the default wineprefix, so please revisit those steps.

With Studio open, you will be directed towards the Setting menu. Please *deselect* "Disable Hardware Rendering!" Having this selected will cause Studio to crash when browsing game content or editing CC.

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

Setting up Blender is a more complicated process, and one that I haven't entirely cracked yet.

You need to download the Windows Portable (WinPort) version from the Blender site (the one that comes in a zip file) and *not* the Linux version (Studio doesn't know what to do with it) or the installer (that is a .msi and won't actually install.) I put mine in my "Applications" folder alongside XML Extractor and Finder and my other random applications.

Once the WinPort version of Blender has been put in an appropriate home, you'll need to manually install several files into the 4.5 folder. The files and instructions can be found here on the official Sims 4 Studio site: https://sims4studio.com/thread/38676/blender-install-add-ons-solved

Once Blender is set up, give Studio the filepath to the installation as you normally would.

I'm not sure why, but at least as of right now it appears you need to open your exported .blend file directly in the WinPort version of Blender and *not* use the shortcut to default opener trick I described above, as that results in Blender loading a cube and not your exported mesh. I'll update this tutorial if I find a way around that.

Many CC users will also use PhotoShop for a number of steps and processes. PhotoShop does not work on Linux, and generally people who want to use Linux will migrate to GIMP or Krita instead, with GIMP the most common destination.

However, PhotoShop users may experience some challenges adjusting to the GIMP UX and getting it to do what they want it to do. There is a project called "PhotoGIMP" that allows users to mod GIMP so it's closer to PhotoShop. This project can be found here: https://photogimp.com/

I hope this tutorial was helpful to you, or at least informative.

---

Originally posted on the [Creator Musings Discord server](https://discord.com/invite/qxz5Kn5).