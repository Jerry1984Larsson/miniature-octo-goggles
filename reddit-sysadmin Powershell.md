Windows PowerShell (sometimes abbreviated "PS" or PoSH) is Microsoft's task automation framework, consisting of a command-line shell and associated scripting language built on top of .NET Framework and C# under the hood. It **WILL** be the method of managing the Windows (and even Linux and other technologies) in the future. Make no mistake, PowerShell is a game-changer when it comes to administration servers and endpoints.

http://en.wikipedia.org/wiki/Windows_PowerShell

Over the past 9 years, the community has a lot of tenacity to make PowerShell better. Follow powershell.org blogs, podcasts, code challenges (called PowerShell games) and now even Ed Wilson over at windows scripting guy on the MS blog does Mini PowerShell games...

Become part of the community of PowerShell users in your area or at least on the IRC chat.

...

In order to understand Powershell, it is best to: learn how to learn, and learn PowerShell by doing PowerShell.

Here is a 4 hour session which will show you how PS works "under the hood" and also give you enough knowledge to get started using Powershell:
http://www.youtube.com/watch?v=-Ya1dQ1Igkc

Microsoft Virtual Academy has two sessions of PowerShell learning and they are free. Sign up with MVA and follow along with the instructors.:

https://mva.microsoft.com/en-us/training-courses/getting-started-with-powershell-3-0-jump-start-8276

https://mva.microsoft.com/en-US/training-courses/advanced-tools-scripting-with-powershell-30-jump-start-8277

One of our [/r/sysadmin](https://www.reddit.com/r/sysadmin) users [/u/ramblingcookiemonster](https://www.reddit.com/u/ramblingcookiemonster) frequently posts here as well as the [/r/powershell](https://www.reddit.com/r/powershell) sub. Follow him as well as the resources on his site: http://ramblingcookiemonster.wordpress.com/

[/u/ramblingcookiemonster](https://www.reddit.com/u/ramblingcookiemonster) has since moved his site to github. You can reach the new site at: http://ramblingcookiemonster.github.io/

The word 'cmdlet' is unique to PowerShell and googling or binging it is your friend. It's encouraged to search for a script on typical locations like codeplex or other repositories for functions that may do what you want (why reinvent the wheel?)

**How Powershell differs from UNIX shells?**

PowerShell is object-oriented whereas UNIX is text based.

Both PS and UNIX shells use a mechanism called "pipes" to pass the results of one function, Cmdlet, or program to the input of another.

However on UNIX the base format for the data passed between programs is a string, whereas within PS the base format is an object. This is where PS really has a lot of power and can pass objects from one cmdlet to another.

A string is a series of characters of unknown length. An object might also be a string, but it can also be any other format which inherits from the object data type.

The object data type has the following built in functions:

|          Name          |                         Description                          |
| :--------------------: | :----------------------------------------------------------: |
|         Object         |                         Constructor                          |
|     Equals(Object)     | Determines whether the specified object is equal to the current object. |
| Equals(Object, Object) | Determines whether the specified object instances are considered equal. |
|        Finalize        | Allows an object to try to free resources and perform other cleanup operations before it is reclaimed by garbage collection. |
|      GetHashCode       |       Serves as a hash function for a particular type.       |
|        GetType         |            Gets the Type of the current instance.            |
|    MemberwiseClone     |        Creates a shallow copy of the current Object.         |
|    ReferenceEquals     | Determines whether the specified Object instances are the same instance. |
|        ToString        |     Returns a string that represents the current object.     |



**PowerShell Shothand Notation**

In Powershell, there are a two useful aliases which are often used in one-line PowerShell commands. These tend to show up in online discussions and can be confusing if you are not used to them:

| Short-hand Notation |      Meaning      |                         Description                          |
| :-----------------: | :---------------: | :----------------------------------------------------------: |
|          %          |  ForEach-Object   | The ForEach-Object commandlet receives a collection of objects from the pipeline and then iterates through that collection |
|          ?          |   Where-Object    | The Where Object receives a collection of objects from the command line, filter it based on the given criteria and then returns that filtered collection |
|         $_          | Pipeline Variable | When using either of the above, this notation is used to reference the instance of the variable. |