---
layout: post
title: "Forensic Analysis of Tor on Windows"
date: 2013-05-20 11:49
comments: true
categories: torproject
---

As part of a deliverable for two Tor Project sponsors ([Sponsor J](https://trac.torproject.org/projects/tor/wiki/org/sponsors/SponsorJ), [Sponsor L](https://trac.torproject.org/projects/tor/wiki/org/sponsors/SponsorL)), I have been working on a forensic analysis of the Tor Browser Bundle. In this three part series, I will summarize the most interesting or significant traces left behind after using the bundle, deleting it, and then shutting down the computer. Part one [covered](http://blog.encrypted.cc/blog/2013/04/12/forensic-analysis-of-tor-on-linux/) Debian Linux ([#8166](https://trac.torproject.org/projects/tor/ticket/8166)), this part will cover Windows 7 ([#6845](https://trac.torproject.org/projects/tor/ticket/6845)), and part three will cover OS X 10.8 ([#6846](https://trac.torproject.org/projects/tor/ticket/6846)). 

### Process

I set up a virtual machine with a fresh install of Windows 7, logged in
with the default admin account, installed available updates, and shut it
down cleanly. I connected the virtual drive to another virtual machine,
used *hashdeep* to compute hashes for every file on the drive, and then
*rsync* to copy all the files over to an external drive.

After having secured a copy of the clean virtual machine, I rebooted the
system, connected an external drive, and copied the Tor Browser Bundle
(version 2.3.25-6, 64-bit) from the external drive to the *Desktop*. I
extracted the package archive by clicking on the file, then started the
Tor Browser Bundle by going into the *Tor Browser* folder and clicking
on *Start Tor Browser.exe*.

Once the Tor Browser was up and running, I browsed to a few pages, read
a few paragraphs here and there, clicked on a few links, and then shut
it down by closing the Tor Browser and clicking on the *Exit*-button in
Vidalia. The Tor Browser did not crash and I did not see any error
messages. I deleted the Tor Browser folder and the package archive by
moving the folder and the archive into the *Recycle Bin*, right-clicking
on it and choosing *Empty Recycle Bin*.

I repeated the steps with hashdeep and rsync to create a copy of
the tainted virtual machine. I also used
[Noriben](http://www.thebaskins.com/main/component/content/article/15-work/60-noriben-your-personal-portable-malware-sandbox),
written by Brian Baskin, to create a report of everything the Tor
Browser Bundle did while it was running.

### Results

Using hashdeep, I compared the hashes from the tainted virtual machine
against the hashes from the clean virtual machine: [256 files](http://blog.encrypted.cc/files/windows_changed_files.txt) 
have hashes that do not match any of the hashes in the clean set.
Additionally, the [Noriben output](http://blog.encrypted.cc/files/noriben.txt) 
shows the Tor Browser Bundle create, edit, and remove a bunch of files.

I have sorted the most interesting findings into different groups,
depending on the location, how they were created, and so on.
Windows 7 has [built-in symbolic links](http://msdn.microsoft.com/en-us/library/3ak841sy.aspx#isolated_storage_locations) 
designed for backward compatibility, which is why Noriben and hashdeep
list the same files in different locations.

#### Prefetch

Windows keeps track of the way the system starts and which programs the
user commonly opens. This information is saved as a number of small
files in the prefetch folder:

 * C:\Windows\Prefetch\START TOR BROWSER.EXE-F5557FAC.pf
 * C:\Windows\Prefetch\TBB-FIREFOX.EXE-350502C5.pf
 * C:\Windows\Prefetch\TOR-BROWSER-2.3.25-6\_EN-US.EX-1354A499.pf
 * C:\Windows\Prefetch\TOR.EXE-D7159D93.pf
 * C:\Windows\Prefetch\VIDALIA.EXE-5167E0BC.pf

The following cache files are most likely similar to prefetch files and
might contain traces of the Tor Browser Bundle:

 * C:\Users\runa\AppData\Local\Microsoft\Windows\Caches\cversions.1.db
 * C:\Users\runa\AppData\Local\Microsoft\Windows\Caches\{AFBF9F1A-8EE8-4C77-AF34-C647E37CA0D9}.1.ver0x0000000000000006.db
 * C:\Windows\AppCompat\Programs\RecentFileCache.bcf

I have created [#8916](https://trac.torproject.org/projects/tor/ticket/8916) for this issue.

#### SetupAPI

SetupAPI and the Plug and Play (PnP) manager write entries to
*SetupAPI.dev.log* and *SetupAPI.app.log* about operations that install
devices and drivers. The following files contain information about the
attached external drive:

 * C:\Windows\inf\setupapi.dev.log
 * C:\Windows\System32\DriverStore\FileRepository\usbstor.inf\_amd64\_ neutral\_0725c2806a159a9d\usbstor.PNF

#### Thumbnail Cache

Windows stores thumbnails of graphics files, and certain document and
movie files, in Thumbnail Cache files. The following files contain the
Onion Logo icon associated with the Tor Browser Bundle:

 * C:\Users\Runa\AppData\Local\Microsoft\Windows\Explorer\thumbcache\_32.db
 * C:\Users\Runa\AppData\Local\Microsoft\Windows\Explorer\thumbcache\_96.db
 * C:\Users\Runa\AppData\Local\Microsoft\Windows\Explorer\thumbcache\_256.db

Other Thumbnail Cache files, such as *thumbcache\_1024.db*,
*thumbcache\_sr.db*, *thumbcache\_idx.db*, and *IconCache.db*, may also
contain the Onion Logo icon. I have created
[#8921](https://trac.torproject.org/projects/tor/ticket/8921) for this issue.

#### Windows Defender

Windows Defender is the default anti-virus software on Windows 7.
Windows Defender will write log files to the following location:

 * C:\ProgramData\Microsoft\Windows Defender\Support\

The log files will contain traces of the Tor Browser Bundle if Windows
Defender ever decides to flag it as malware. This is true for any
anti-virus software.

#### Windows Error Reporting (WER)

Windows Error Reporting (WER) captures and logs information about
software crashes and other issues. I found information about the
attached external drive in the following file:

 * C:\Users\runa\AppData\Local\Microsoft\Windows\WER\ReportQueue\NonCritical\_x64\_ 84cd279a5e83221bfa7edcb36665592c1974e4\_cab\_0b21673a/DMI671B.tmp.log.xml

The logs will probably contain traces of the Tor Browser Bundle if any
part of the bundle, such as the Tor Browser or Vidalia, ever hangs or
crashes.

#### Windows Event Log

The following two event logs contain information about the attached
external drive:

 * C:\Windows\System32\winevt\Logs\Application.evtx
 * C:\Windows\System32\winevt\Logs\System.evtx

#### Windows Paging File

Microsoft Windows uses a paging file, called *pagefile.sys*, to store
frames of memory that do not currently fit into physical memory. The
file *C:\pagefile.sys* contains information about the attached external
drive, as well as the filename for the Tor Browser Bundle executable. I
have created [#8918](https://trac.torproject.org/projects/tor/ticket/8918) for this issue.

#### Windows Registry

The Windows Registry is a databse that stores various configuration
settings and options for the operating system. *HKEY\_CURRENT\_USER*,
abbreviated *HKCU*, stores settings that are specific to the currently
logged-in user. Each user's settings are stored in files called
*NTUSER.DAT* and *UsrClass.dat*.

The path to the Tor Browser Bundle executable is listed in the following
two files:

 * C:\Users\runa\AppData\Local\Microsoft\Windows\UsrClass.dat
 * C:\Users\runa\AppData\Local\Microsoft\Windows\UsrClass.dat.LOG1

I did not find traces of the Tor Browser Bundle in any of the
*NTUSER.DAT* files. I have created
[#8919](https://trac.torproject.org/projects/tor/ticket/8919) for
this issue.

Additionally, the output from Noriben indicates that the Tor Browser
Bundle touches the registry by creating keys and setting values. The
following value points to the Tor Browser Bundle executable on the
attached external drive:

 * [Set Value] Explorer.EXE:1196 > HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\MuiCache\E:\tor-browser-2.3.25-6\_en-US.exe  =  7z SFX

The output also makes it look like the Tor Browser Bundle adds the following keys and values:

 * [Set Value] tbb-firefox.exe:1124 > HKCU\Software\Classes\Local Settings\MuiCache\11\52C64B7E\LanguageList  =  en-US, en
 * [CreateKey] tbb-firefox.exe:1124 > HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}
 * [CreateKey] tbb-firefox.exe:1124 > HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}\Count
 * [CreateKey] tbb-firefox.exe:1124 > HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}
 * [CreateKey] tbb-firefox.exe:1124 > HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}\Count

I found that these keys and values are present on a clean Windows 7 system
where the Tor Browser Bundle has never been used. I also downloaded and tested
the German version of the Tor Browser Bundle to make sure that the
*LanguageList* value does not represent the language of the Tor Browser Bundle.

#### Windows Search

Windows Search, which is enabled by default, builds a full-text index of
files on the computer. One component of Windows Search is the Indexer,
which crawls the file system on initial setup, and then listens for file
system notifications to index changed files. Windows Search writes a number of
files to the following location:

 * C:\ProgramData\Microsoft\Search\Data\Applications\Windows\

I have not found a way to read the Windows Search database files, but I
would say it is likely that Windows Search picked up the Tor Browser
Bundle at some point. I have created
[#8920](https://trac.torproject.org/projects/tor/ticket/8920) for this issue.
