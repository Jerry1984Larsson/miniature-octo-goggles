For several years we’ve used a basic data backup program that exploits the archive bit on files to determine if they need to be backed up or not. We run a weekly full backup with daily differentials. After the weekly backup is run, the archive bit is reset on all files, ready for the next week.

Except it isn’t.

It turns out that if the file has both the system and hidden bit set, you can’t change any of the attributes. This is apparently [a known issue that goes right back to the attrib command back in the MS-DOS days](https://jeffpar.github.io/kbarchive/kb/081/Q81361/).

The end result is that our differential backups end up full of seemingly empty directories that simply don’t need to be there. The usual culprit is a hidden desktop.ini or thumbs.db file.

The simple solution would of course be to run the attrib -a /s /d command on the drive after the weekly backup has completed. But as we now know, that doesn’t work:

[![](https://greiginsydney.com/wp-content/uploads/2020/10/xNotResettingHiddenFile-300x141.png.pagespeed.ic.jc_rAKcBa5.webp)](https://greiginsydney.com/wp-content/uploads/2020/10/NotResettingHiddenFile.png)

### PowerShell to the Rescue!

  
Here are two helpful PowerShell commands. The first can be used to reveal files with their archive bit set, prior to you clearing it.

#### Find Files to Archive

Use this command to find all files with the Archive bit set. Delete “-recurse” if you DON’T want it to look into sub-folders.

Get-ChildItem <Drive\\Folder\\File> -recurse -force | where {(Get-ItemProperty $\_.fullname).attributes -BAND \[io.fileattributes\]::archive}

#### Clear the Archive Bit

Here’s a so-called one-liner that will do the job:

Get-ChildItem <Drive\\Folder\\File> -recurse -force | foreach {Set-ItemProperty -path $\_.FullName -Name attributes -value ((Get-ItemProperty $\_.fullname).attributes -BAND (\[io.fileattributes\]::hidden + \[io.fileattributes\]::readonly) + \[io.fileattributes\]::normal + \[io.fileattributes\]::system) }

Here’s what it’s doing, step by step:

Get-ChildItem <Drive\\Folder\\File>> -recurse -force |
foreach {Set-ItemProperty -path $\_.FullName -Name attributes -value 
((Get-ItemProperty $\_.fullname).attributes -BAND (\[io.fileattributes\]::hidden + \[io.fileattributes\]::readonly) + \[io.fileattributes\]::normal + \[io.fileattributes\]::system) }

1.  Return all the files in the nominated path, even the hidden ones
2.  For each file, we’ll set its ‘attributes’ property with a value of…
3.  the result of a Binary AND (‘BAND’) of the file’s current attributes, and a grouping of all the writeable attributes permitted – EXCEPT for Archive.

The end result of the final step – the AND – is that if any of the other attributes (System, Normal, ReadOnly and Hidden) are present, they’ll be retained, but if Archive is set it will be dropped.

As always, re-run the ‘find files’ commandlet upon completion to confirm the outcome is as expected.

### References

Thank you Scripting Guy Ed Wilson: [Use PowerShell to Toggle the Archive Bit on Files](https://devblogs.microsoft.com/scripting/use-powershell-to-toggle-the-archive-bit-on-files/).

### Revision History

6th October 2020. This is the initial publication.

  – G.