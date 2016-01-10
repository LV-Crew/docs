---
layout: page
title: "CleanerML"
category: doc
date: 2015-09-25 23:35:38
order: 7
---

**CleanerML** is a simple yet powerful markup language for writing cleaners. Most of BleachBit's cleaners are written in CleanerML, and you can write your own cleaners in CleanerML too.

You can think of it as writing XML to delete files, but it is more powerful than that. CleanerML's features include:

* Familiar XML
* Open standard
* Cleaners
* Delete, truncate, or shred files
* Delete Windows registry keys and named values
* Vacuum SQLite databases
* Clean .ini configuration files
* Clean JSON configuration files
* Find files by glob, walking a tree, or 'deep scan'
* Refine a file search with a regular expression
* Export to gettext for translation using standard tools such as Launchpad
* Operating system detection to discard OS-specific cleaners (such as Winamp) at runtime
* XSD (XML Schema Definition) for validation

### Storage

During application startup, BleachBit looks for CleanerML files in a few standard locations:

*   ```/usr/share/bleachbit/cleaners/``` on Linux
*   ```~/.config/bleachbit/cleaners/``` on Linux
*   ```share/cleaners/``` relative to the Python script on Linux (useful for running BleachBit from source without installation
*   ```share\cleaners\``` relative to the BleachBit executable on Windows which typically translates to ```c:\program files\bleachbit\share\cleaners```  
    Warning: This directory is deleted when BleachBit is updated or uninstalled.
*   ```%APPDATA%\BleachBit\cleaners\``` on Windows which typically translates to
    *   Windows XP: ```C:\Documents and Settings\(username)\Application Data\BleachBit\Cleaners\```
    *   Windows Vista/7: ```C:\Users\(username)\AppData\Roaming\BleachBit\Cleaners\```

Most of these locations are also scanned for [winapp2.ini](/documentation/winapp2_ini) files, but you may only use one winapp2.ini file.

### Learning CleanerML

To learn CleanerML so you can write your own cleaner, read these resources:

*   [Example cleaner](https://github.com/az0/bleachbit/blob/master/doc/example_cleaner.xml) with many annotations
*   [Cleaners that come standard with BleachBit](https://github.com/az0/bleachbit/tree/master/cleaners)
*   [Bonus cleaners](https://github.com/az0/cleanerml)
*   [XSD (XML Schema Definition)](https://github.com/az0/bleachbit/blob/master/doc/cleaner_markup_language.xsd) used for validation

### Finding files to delete

1.  Run the application you want to clean.
    *   In the applications' preferences, turn on all logging (if applicable). For example, by default Pidgin turns off chat logs.
    *   Use all the features of the application to try to make it generate as many files as it can. For example, in Nexuiz (a game) you must play a multiplayer game with a new map to cause the game to download the map into its cache. Many Nexuiz multiplayer games don't download maps.
2.  Discover where the application stores its file. Assume your application is called Firefox: you could use the following commands to begin the find its files.  
    Linux: `ls -d ~/.* | tail -n+3 | xargs -I '{}' find '{}' | grep -i firefox`  
    Windows: `dir /s /b $USERPROFILE | find /i "firefox"`
3.  In some cases not all files appear using those commands. Perhaps the application checks for a file which doesn't often exist, or it writes a file but deletes it a moment later. In these cases use strace (for Linux) or [Process Monitor](http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx) (for Windows) to find these files. A standard invokation of strace (assuming you are launching Firefox) is: `strace -f -e trace=open,stat64,lstat64,access,mkdir,unlink,rename,readlink firefox &> /tmp/firefox.log grep -iE "(cache|log|tmp|$HOME)" /tmp/firefox.log | sort | uniq | less`
4.  Search for registry entries in Windows: typically they are under ```HCKU\Software\(app name)```.
5.  It's not strictly necessary, but it's nice if you put your cleaner in the ```cleaners``` directory of the BleachBit source and run ```make tests``` (to check it against the XSD) and ```make pretty``` (to reformat the XML).

### Matching files

CleanerML allows several ways to match files:

*   **file**: matches a single file.
*   **glob**: matches one or more files with a simple pattern. See the Python documentation on [glob](http://docs.python.org/library/glob.html).
*   **walk.files**: matches all files under a directory (but does not match directories).
*   **walk.all**: matches all and directories files under a directory.
*   **deep**: queues a deep scan

What is the difference between a **deep** and **walk.files**? Deep scan expect matches to be loosely scattered (such as Thumbs.db), but **walk.files** expects to match most files under that directory (such as Firefox's cache). To improve performance, BleachBit combines deep scans for the same directory (such as all deep scans for $HOME). In the future, BleachBit may allow the user to reconfigure the deep scan directory, so, for example, he can scan a network drive in addition to his home directory.

Any of these methods can be combined with [Python's Perl regular expressions](http://docs.python.org/howto/regex.html#regex-howto) for sophisticated filtering.

For more information, refer to the section [Learning](#learning-cleanerml).

### Environment variables

Instead of hard coding path names, use an environment variable whenever possible because common paths may change depending on user logged in, the version of Windows, and on the user's language. In addition to expanding ```~``` to the user's home directory (such as ```/home/andrew/```), CleanerML expands environment variables. On both Linux and Windows environment variables must be given in the Bash format ```$foo``` instead of the Windows format ```%foo%```.

The most common environment variables are APPDATA, LOCALAPPDATA, PROGRAMFILES, and USERPROFILE. For more information, see Wikipedia's ["Environment variable"](http://en.wikipedia.org/wiki/Environment_variable).

### Sharing your cleaner

Of course you may use your cleaner privately. If you wish to share it with others, see [Contribute Cleaner](http://bleachbit.sourceforge.net/contribute/cleaner).


