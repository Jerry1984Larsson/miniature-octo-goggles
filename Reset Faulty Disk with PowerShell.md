# Reset Faulty Disk with PowerShell 



```powershell
Get-PhysicalDisk | ft FriendlyName, SerialNumber, UniqueId -auto
```



![get disks’ FriendlyNames and UniqueIds ](https://www.minitool.com/images/uploads/news/2021/05/0x00000032/0x00000032-1.png)



Reset the problematic disk by its FriendlyName or UniqueId. Below are examples for reset the first hard drive by its FriendlyName and UniqueId respectively. You just need to choose one command order to reset your bad disk with your disk name or ID.

```powershell
Reset-PhysicalDisk - FriendlyName "WDC WD5000AAKX-08U6AA0"
```

```powershell
Reset-PhysicalDisk -UniqueId " 50014E2E0C679D36"
```

If there are more than one bad disks that need resetting, you can rely on the below command to reset them all in one time.



```powershell
$BadDisks = Get-Physicaldisk | Where-Object -FilterScript {$_.HealthStatus -Eq "Unhealthy"}

Foreach ($BadDisk in $BadDisks)

{

  Reset-PhysicalDisk -UniqueId $BadDisk.UniqueId

}
```

