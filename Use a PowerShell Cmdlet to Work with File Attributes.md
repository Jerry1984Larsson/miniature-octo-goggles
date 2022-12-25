![Avatar](https://devblogs.microsoft.com/scripting/wp-content/uploads/sites/29/2019/08/PowerShell_5.0_icon-150x150.png)

January 26th, 2011

**Summary**: Learn how to use the Windows PowerShell cmdlet Set-ItemProperty to work with file attributes.

[![Hey, Scripting Guy! Question](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/q-for-powertip.jpg "Hey, Scripting Guy! Question")](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/q-for-powertip.jpg) Hey, Scripting Guy! I often find myself working with file attributes. Our backup program reads the archive flag, and our users are always creating read-only copies of their spreadsheets. I have an old VBScript script that will manipulate file attributes, but I am surprised that Windows PowerShell does not do this. It seems like it should be simple to have a **Set-FileAttribute** Windows PowerShell cmdlet, but unfortunately it does not seem to exist.

— AD

[![Hey, Scripting Guy! Answer](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/a-for-powertip.jpg "Hey, Scripting Guy! Answer")](https://devblogs.microsoft.com/wp-content/uploads/sites/29/2019/02/a-for-powertip.jpg) Hello AD,

Microsoft Scripting Guy Ed Wilson here. One of the cool things about being in Florida during the winter is actually a warm thing. Today, it is 71 degrees Fahrenheit (21.6 degrees Celsius according to my [conversion module](http://blogs.technet.com/b/heyscriptingguy/archive/2010/02/21/hey-scripting-guy-february-21-2010.aspx)), and it is sunny. It was such a lovely day that Dr. Scripto decided to head down to the harbor to watch the boats coming in and out.

[![Image of Dr. Scripto watching boats come and go in the harbor](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/7065.HSG-1-26-11-1_thumb_7531AB3B.jpg "Image of Dr. Scripto watching boats come and go in the harbor")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/prod.evol.blogs.technet.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/76/18/metablogapi/3771.HSG-1-26-11-1_253BFCBD.jpg)

AD, I will leave Dr. Scripto at the waterfront to work on his tan (he really needs to get out more), and I will tackle your question. In September of last year, I wrote [How Can I Unlock a Read-Only File, Edit it, and Make it Read-Only Again](http://blogs.technet.com/b/heyscriptingguy/archive/2009/09/24/hey-scripting-guy-september-24-2009.aspx)? In that post, which was awesome, I worked specifically with the read-only attribute. In this article, I will look at a different way of working with file attributes.

AD, you said you wish that Windows PowerShell had a **Set-FileAttribute** cmdlet. In fact, it does. It is called **Set-ItemProperty**. Most people use the **Set-ItemProperty** cmdlet when working with the registry provider, but the **Set-ItemProperty** can work with any provider that provides access to item properties. I use the **Get-PSProvider** cmdlet to detail the Windows PowerShell providers that are currently available to me on my laptop. If I load additional modules or snap-ins, it is possible that additional providers are exposed. Therefore, it is always a good idea to use **Get-PSProvider** after loading a module or snap-in. The results of **Get-PSProvider** with no snap-ins or modules loaded appears here:

> PS C:\\> Get-PSProvider
>
> Name Capabilities Drives
>
> —- ———— ——
>
> WSMan Credentials {WSMan}
>
> Alias ShouldProcess {Alias}
>
> Environment ShouldProcess {Env}
>
> FileSystem Filter, ShouldProcess {C, D}
>
> Function ShouldProcess {Function}
>
> Registry ShouldProcess, Transactions {HKLM, HKCU}
>
> Variable ShouldProcess {Variable}
>
> Certificate ShouldProcess {cert}
>
> PS C:\\>

The **FileSystem** provider works with my C and D drives. To find out what types of item properties are available, I use the **Get-ItemProperty** to retrieve a file from my FSO folder. The syntax of the command is **Get-ItemProperty** and the path to the file. The command and its associated output appear here:

> PS C:\\> Get-ItemProperty -Path C:\\fso\\a.txt
>
> Directory: C:\\fso
>
> Mode LastWriteTime Length Name
>
> —- ————- —— —-
>
> \-a— 12/17/2010 5:08 PM 0 a.txt
>
> PS C:\\>

This is the top-level view of the item properties of the file. To view all of the information that is available, I pipe the results to the **Format-List** cmdlet, and use the force. This command appears here:

> PS C:\\> Get-ItemProperty -Path C:\\fso\\a.txt | Format-list -Property \* -Force
>
> PSPath : Microsoft.PowerShell.Core\\FileSystem::C:\\fso\\a.txt
>
> PSParentPath : Microsoft.PowerShell.Core\\FileSystem::C:\\fso
>
> PSChildName : a.txt
>
> PSDrive : C
>
> PSProvider : Microsoft.PowerShell.Core\\FileSystem
>
> VersionInfo : File: C:\\fso\\a.txt
>
> InternalName:
>
> OriginalFilename:
>
> FileVersion:
>
> FileDescription:
>
> Product:
>
> ProductVersion:
>
> Debug: False
>
> Patched: False
>
> PreRelease: False
>
> PrivateBuild: False
>
> SpecialBuild: False
>
> Language:
>
> BaseName : a
>
> Mode : -a—
>
> Name : a.txt
>
> Length : 0
>
> DirectoryName : C:\\fso
>
> Directory : C:\\fso
>
> IsReadOnly : False
>
> Exists : True
>
> FullName : C:\\fso\\a.txt
>
> Extension : .txt
>
> CreationTime : 8/17/2009 9:28:43 AM
>
> CreationTimeUtc : 8/17/2009 1:28:43 PM
>
> LastAccessTime : 8/22/2009 6:13:01 PM
>
> LastAccessTimeUtc : 8/22/2009 10:13:01 PM
>
> LastWriteTime : 12/17/2010 5:08:38 PM
>
> LastWriteTimeUtc : 12/17/2010 10:08:38 PM
>
> Attributes : Archive
>
> PS C:\\>

Okay, that provides a lot of information, but it does not tell me if I can write to the item properties or not. To see this information, I need to use the **Get-Member** cmdlet. In the place of the previous **Format-List** command, I use **Get-Member**. Because I am only interested in properties, I use the **membertype** parameter and specify **property**. The revised command and its associated output appear here:

> PS C:\\> Get-ItemProperty -Path C:\\fso\\a.txt | Get-Member -MemberType property
>
> TypeName: System.IO.FileInfo
>
> Name MemberType Definition
>
> —- ———- ———-
>
> Attributes Property System.IO.FileAttributes Attributes {get;set;}
>
> CreationTime Property System.DateTime CreationTime {get;set;}
>
> CreationTimeUtc Property System.DateTime CreationTimeUtc {get;set;}
>
> Directory Property System.IO.DirectoryInfo Directory {get;}
>
> DirectoryName Property System.String DirectoryName {get;}
>
> Exists Property System.Boolean Exists {get;}
>
> Extension Property System.String Extension {get;}
>
> FullName Property System.String FullName {get;}
>
> IsReadOnly Property System.Boolean IsReadOnly {get;set;}
>
> LastAccessTime Property System.DateTime LastAccessTime {get;set;}
>
> LastAccessTimeUtc Property System.DateTime LastAccessTimeUtc {get;set;}
>
> LastWriteTime Property System.DateTime LastWriteTime {get;set;}
>
> LastWriteTimeUtc Property System.DateTime LastWriteTimeUtc {get;set;}
>
> Length Property System.Int64 Length {get;}
>
> Name Property System.String Name {get;}
>
> PS C:\\>

That is what I was looking for. Notice that in the definition column some of the properties such as **Attributes** are listed as **{get;set;}** and others such as **Directory** are listed as **{get;}**. Well, if a property is **get;set** then it is read-write–in other words, I can modify the value. If a property is only **get**, it is read-only.

The other thing that is useful from the Definition column is the data type that the property accepts. For example, the **CreationTime** property accepts a **System.DateTime** object. (I have written a number of [posts about working with dates and times in Windows PowerShell](http://blogs.technet.com/b/heyscriptingguy/archive/tags/windows+powershell/dates+and+times/) on the Hey, Scripting Guy! Blog.) I can obtain a **System.DateTime** object from the **Get-Date** cmdlet as shown here:

> PS C:\\> (Get-Date).gettype()
>
> IsPublic IsSerial Name BaseType
>
> ——– ——– —- ——–
>
> True True DateTime System.ValueType
>
> PS C:\\>

On the other hand, the **Attributes** property accepts an enumeration value called **System.Io.FileAttributes** (I have written a number of Hey, Scripting Guy! Blog [posts about working with enumerations](http://blogs.technet.com/b/heyscriptingguy/archive/tags/windows+powershell/getting+started/enum/) as well.) To see the enumeration values from **System.Io.FileAttributes**, I can place the class name in square brackets and use **Get-Member**, as shown here:

> PS C:\\> \[System.Io.FileAttributes\] | Get-Member -Static -MemberType property
>
> TypeName: System.IO.FileAttributes
>
> Name MemberType Definition
>
> —- ———- ———-
>
> Archive Property static System.IO.FileAttributes Archive {get;}
>
> Compressed Property static System.IO.FileAttributes Compressed {get;}
>
> Device Property static System.IO.FileAttributes Device {get;}
>
> Directory Property static System.IO.FileAttributes Directory {get;}
>
> Encrypted Property static System.IO.FileAttributes Encrypted {get;}
>
> Hidden Property static System.IO.FileAttributes Hidden {get;}
>
> Normal Property static System.IO.FileAttributes Normal {get;}
>
> NotContentIndexed Property static System.IO.FileAttributes NotContentIndexed {get;}
>
> Offline Property static System.IO.FileAttributes Offline {get;}
>
> ReadOnly Property static System.IO.FileAttributes ReadOnly {get;}
>
> ReparsePoint Property static System.IO.FileAttributes ReparsePoint {get;}
>
> SparseFile Property static System.IO.FileAttributes SparseFile {get;}
>
> System Property static System.IO.FileAttributes System {get;}
>
> Temporary Property static System.IO.FileAttributes Temporary {get;}
>
> PS C:\\>

I can also use the static **GetValues** method from the **system.enum** .NET Framework class (I can leave off the word _system_ when calling the class). This appears here:

> PS C:\\> \[enum\]::GetValues(\[system.io.fileattributes\])
>
> ReadOnly
>
> Hidden
>
> System
>
> Directory
>
> Archive
>
> Device
>
> Normal
>
> Temporary
>
> SparseFile
>
> ReparsePoint
>
> Compressed
>
> Offline
>
> NotContentIndexed
>
> Encrypted
>
> PS C:\\>

If I want to see the specific value associated with a particular enumeration value, I use the **value\_\_** property (keep in mind that is a double underscore at the end of the word _value_). This technique appears here:

> PS C:\\> \[system.io.fileattributes\]::ReadOnly.Value\_\_
>
> 1
>
> PS C:\\>

Using the **Get-EnumValues** function from my [Enumerations and Values Weekend Scripter post](http://blogs.technet.comhttps//devblogs.microsoft.com/scripting/hey-scripting-guy-weekend-scripter-enumerations-and-values/), I obtain the following list of enumerations and their associated values.

> Name Value
>
> —- —–
>
> Offline 4096
>
> NotContentIndexed 8192
>
> Device 64
>
> Directory 16
>
> Normal 128
>
> ReparsePoint 1024
>
> Archive 32
>
> Encrypted 16384
>
> SparseFile 512
>
> System 4
>
> Temporary 256
>
> Hidden 2
>
> ReadOnly 1
>
> Compressed 2048

Ok, so far so good. If the attributes were stored as a hash table or as an array, things would be easy. Alas, they are not. They are stored as an old-fashioned bitmask value.

Therefore, to work with them, we need to use old-fashioned techniques, such as the sort thing you learned when you were taking Boolean algebra back at the university. If you are little rusty on your Boolean algebra, you may want to refer to [this Hey, Scripting Guy! Blog](http://blogs.technet.com/b/heyscriptingguy/archive/2004/10/19/how-can-i-change-a-read-only-file-to-a-read-write-file.aspx) post. It is sort of like the [Karate Kid](http://en.wikipedia.org/wiki/The_karate_kid) and “wax on, wax off” only different.

To determine if a file is read-only or not, I need to perform a bitwise AND operation. Therefore, I will use the –BAND operator. Let’s see how that would work out. The file attributes of the a.txt file are **ReadOnly** and **Archive**. This is shown here:

> PS C:\\> (Get-ItemProperty -Path C:\\fso\\a.txt).attributes
>
> ReadOnly, Archive
>
> PS C:\\>

I want to know what that value is so that I can use the **Value\_\_** property. However, I also want to know what that value is in binary, so I will use the **\[convert\]** class. This technique appears here:

> PS C:\\> (Get-ItemProperty -Path C:\\fso\\a.txt).attributes.Value\_\_
>
> 33
>
> PS C:\\> \[convert\]::ToString(33,2)
>
> 100001
>
> PS C:\\>

Everywhere a file attribute is turned on, a 1 appears. Therefore, the value 32 (archive) appears in the sixth position, and read-only (value of 1) appears in the first position. This behaves in a similar way that figuring out subnet masks works. In the table following this paragraph, the first row is the decimal value of the position. In the second row, is the binary representation of the number 33 that I obtained by using the **convert** class earlier. In the second row, notice there is a 1 in the 32 column, and a 1 in the 1 column. The third row is the binary value of the number 1 (the value of the read-only enumeration). The fourth row represents the results of performing a binary AND (BAND) operation. When performing a Binary AND operation, the rules are thus: 1 –BAND 1 = 1, 1 –BAND 0 = 0, 0 –BAND 0 = 0.

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

0

0

0

0

0

1

I can use this information to determine if a file has the archive bit or the read-only bit set. This is illustrated here:

> PS C:\\> (Get-ItemProperty C:\\fso\\a.txt).attributes -band \[io.fileattributes\]::Archive
>
> 32
>
> PS C:\\> (Get-ItemProperty C:\\fso\\a.txt).attributes -band \[io.fileattributes\]::ReadOnly
>
> 1
>
> PS C:\\>

AD, that is all there is to using **Set-ItemProperty** to retrieve file attributes. Neglected Cmdlet Week will continue [tomorrow when I will talk about using **Set-ItemProperty** to modify file attributes](http://blogs.technet.com/b/heyscriptingguy/archive/2011/01/27/use-powershell-to-toggle-the-archive-bit-on-files.aspx).

I invite you to follow me on [Twitter](http://bit.ly/scriptingguystwitter) and [Facebook](http://bit.ly/scriptingguysfacebook). If you have any questions, send email to me at [scripter@microsoft.com](mailto:scripter@microsoft.com), or post your questions on the [Official Scripting Guys Forum](http://bit.ly/scriptingforum). See you tomorrow. Until then, peace.

**Ed Wilson, Microsoft Scripting Guy**
