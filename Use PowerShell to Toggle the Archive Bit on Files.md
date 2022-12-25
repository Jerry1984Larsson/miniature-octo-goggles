![Avatar](https://devblogs.microsoft.com/scripting/wp-content/uploads/sites/29/2019/08/PowerShell_5.0_icon-150x150.png)

January 27th, 2011

**Summary**: Learn how to use Windows PowerShell to toggle file attributes such as the archive bit.

[![Hey, Scripting Guy! Question](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/q-for-powertip.jpg "Hey, Scripting Guy! Question")](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/q-for-powertip.jpg) Hey, Scripting Guy! I have a number of files that have had the archive bit flipped. Unfortunately, our backup program relies on the archive bit to determine if it has backed up a file recently or if a file has changed and is in need of backup again. What I need is a way to flip all of the archive bits in a folder.

— BW

[![Hey, Scripting Guy! Answer](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/a-for-powertip.jpg "Hey, Scripting Guy! Answer")](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/a-for-powertip.jpg) Hello BW,

Microsoft Scripting Guy Ed Wilson here. Backup programs. Hmmm, I used to know a person who claimed he had an ideal business model. He said he would advertise extremely low-cost backup tape storage. The backup tapes would be stored in a big hole in the ground. When someone requested a tape from “storage,” they would be given a blank tape. This model would work, he said, because no one ever tests backup tapes, and no one ever practices a file restore. Luckily, this person never went into business, and never implemented the “low-cost storage” solution.

BW, I have seen the problem you mention occur when someone runs a scheduled daily backup to perform a custom ad hoc backup, instead of creating a custom on-demand job that does not flip archive bits. When this happens, backups get out of sequence, and it can be a real mess.

> In [yesterday’s Hey, Scripting Guy! Blog post](http://blogs.technet.com/b/heyscriptingguy/archive/2011/01/26/use-a-powershell-cmdlet-to-work-with-file-attributes.aspx), I began a discussion of using the **Set-ItemProperty** Windows PowerShell cmdlet to work with archive bits on a file.

The FSO library on my laptop contains a mixture of files. Some have the archive bit set and others do not. Some are compressed, others encrypted, and still others read-only. The folder is shown in the following image.

[![Image of FSO library on Ed's laptop](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/3247.HSG-1-27-11-1_thumb_4EA1ECEB.jpg "Image of FSO library on Ed's laptop")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/2158.HSG-1-27-11-1_27E3FC78.jpg)

The Get-FilesWithArchiveBitSet.ps1 script will go through a folder and report files that have the archive bit set. It does not matter if there are additional file attributes; the script will detect the presence of the archive bit. The use of the **io.fileattributes** enumeration was discussed in yesterday’s Hey, Scripting Guy! Blog post.

The key to the script is the use of the **If** statement to perform a bitwise AND operation on the file attributes:

> If((Get-ItemProperty -Path $file.fullname).attributes -band $attribute)

If the attribute is present, it is reported in green. If the attribute is not present, it is reported in blue:

> { Write-Host -ForegroundColor green \`
>
> "$file.fullname has the $attribute bit set" }
>
> ELSE
>
> { Write-host -ForegroundColor blue \`
>
> "$file.fullname does not have the $attribute bit set"}

The complete script is shown here.

> Get-FilesWithArchiveBitSet.ps1
>
> $path = "C:\\fso"
>
> $files = Get-ChildItem -Path $path -Recurse
>
> $attribute = \[io.fileattributes\]::archive
>
> Foreach($file in $files)
>
> {
>
> If((Get-ItemProperty -Path $file.fullname).attributes -band $attribute)
>
> { Write-Host -ForegroundColor green \`
>
> "$file.fullname has the $attribute bit set" }
>
> ELSE
>
> { Write-host -ForegroundColor blue \`
>
> "$file.fullname does not have the $attribute bit set"}
>
> } #end Foreach

When the Get-FilesWithArchiveBitSet.ps1 script runs, the output appears that is shown in the following image:

[![Image of script output](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/7457.HSG-1-27-11-2_thumb_6EACD3DB.jpg "Image of script output")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/8015.HSG-1-27-11-2_1DBF3580.jpg)

Now that we understand the basic process, let’s add one additional concept. To toggle the archive bit, I need to use a Bitwise Exclusive Or (BXOR) operator.

In the table that appears here, the first row is the decimal value of the position. In the second row is the binary representation of the number 33 that I obtained by using the **convert** class. In the second row, notice there is a 1 in the 32 column, and a 1 in the 1 column. The third row is the binary value of the number 1 (the value of the read-only enumeration). The fourth row represents the results of performing a bitwise Exclusive OR (BXOR) operation. When performing a BXOR operation, the rules are thus:

1 –BXOR 1 = 0, 1 –BXOR 0 = 1, 0 –BXOR 0 = 0.

128

64

32

16

8

4

2

1

0

0

1

0

0

0

0

1

0

0

0

0

0

0

0

1

0

0

1

0

0

0

0

0

I can use this information to determine the attributes other than the archive bit of a file. This is illustrated here. Notice that when I use **–bxor** on the file attributes, and **–bxor** with the archive bit, what remains is the **ReadOnly** attribute.

> PS C:\\> (Get-ItemProperty C:\\fso\\a.txt).attributes
>
> ReadOnly, Archive
>
> PS C:\\> (Get-ItemProperty C:\\fso\\a.txt).attributes -bxor \[io.fileattributes\]::Archive
>
> 1
>
> PS C:\\>

Armed with this information, I create a script that will toggle the archive bit of a file. If the archive bit exists, it is removed; if it does not exist, it is added to the file. The Set-FilesWithArchiveBit.ps1 script is shown here.

> Set-FilesWithArchiveBit.ps1
>
> $path = "C:\\fso"
>
> $files = Get-ChildItem -Path $path -Recurse
>
> $attribute = \[io.fileattributes\]::archive
>
> Foreach($file in $files)
>
> {
>
> If((Get-ItemProperty -Path $file.fullname).attributes -band $attribute)
>
> {
>
> "$file.fullname has the $attribute bit set, removing the bit."
>
> Set-ItemProperty -Path $file.fullname -Name attributes \`
>
> \-Value ((Get-ItemProperty $file.fullname).attributes -BXOR $attribute)
>
> "New value of $file.Fullname attributes"
>
> (Get-ItemProperty -Path $file.fullname).attributes
>
> }
>
> ELSE
>
> {
>
> Write-host -ForegroundColor blue \`
>
> "$file.fullname does not have the $attribute bit set, setting the bit."
>
> Set-ItemProperty -Path $file.fullname -Name attributes \`
>
> \-Value ((Get-ItemProperty $file.fullname).attributes -BXOR $attribute)
>
> "New value of $file.Fullname attributes"
>
> (Get-ItemProperty -Path $file.fullname).attributes
>
> }
>
> } #end Foreach

The Set-FilesWithArchiveBit.ps1 script is similar to the earlier script except that the **Set-ItemProperty** Windows PowerShell cmdlet is used to change the value of the attributes of the file. To do this, I use **Get-ItemProperty** to retrieve the file attributes, and then I use BXOR to perform a bitwise exclusive OR operation with my chosen attribute. This portion of the code is shown here:

> Set-ItemProperty -Path $file.fullname -Name attributes \`
>
> \-Value ((Get-ItemProperty $file.fullname).attributes -BXOR $attribute)

When the Set-FilesWithArchiveBit.ps1 script runs, the output appears that is shown in the following image:

[![Image of script output](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/8446.HSG-1-27-11-3_thumb_4D89B174.jpg "Image of script output")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/2642.HSG-1-27-11-3_3E364F65.jpg)

One thing to keep in mind about using the **Set-ItemProperty** cmdlet to work with file attributes through the **FileSystem** provider is that it is limited to working with the following attributes: **Archive**, **Hidden**, **Normal**, **ReadOnly**, or **System**. An error will be generated, and the attributes will not be modified if a file is compressed or encrypted.

BW, that is all there is to using the **Set-ItemProperty** cmdlet to toggle the archive bit.

I invite you to follow me on [Twitter](http://bit.ly/scriptingguystwitter) and [Facebook](http://bit.ly/scriptingguysfacebook). If you have any questions, send email to me at [scripter@microsoft.com](mailto:scripter@microsoft.com), or post your questions on the [Official Scripting Guys Forum](http://bit.ly/scriptingforum). See you tomorrow. Until then, peace.

**Ed Wilson, Microsoft Scripting Guy**
