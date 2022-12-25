# **PowerShell - Create a Menu**

```powershell
$MainMenu = {
Write-Host " ***************************"
Write-Host " *           Menu          *"
Write-Host " ***************************"
Write-Host
Write-Host " 1.) Export virtual machines"
Write-Host " 2.) Delete virtual machines"
Write-Host " 3.) Start virtual machines"
Write-Host " 4.) Stop virtual machines"
Write-Host " 5.) Quit"
Write-Host
Write-Host " Select an option and press Enter: "  -nonewline
}
cls
```

```powershell
Do {
cls
Invoke-Command $MainMenu
$Select = Read-Host
Switch ($Select)
    {
    1 {
        Commands to be run if user selects menu option 1
       }
    2 {
        Commands to be run if user selects menu option 2
       }
    3 {
        Commands to be run if user selects menu option 3
       }
    4 {
        Commands to be run if user selects menu option 4
       }
    }
}
While ($Select -ne 5)
```

**Sample**



```powershell
$MainMenu = {
Write-Host " ***************************"
Write-Host " *           Menu          *" 
Write-Host " ***************************" 
Write-Host 
Write-Host " 1.) Export virtual machines" 
Write-Host " 2.) Delete virtual machines"
Write-Host " 3.) Start virtual machines" 
Write-Host " 4.) Stop virtual machines" 
Write-Host " 5.) Quit"
Write-Host 
Write-Host " Select an option and press Enter: "  -nonewline
}
cls

Do { 
cls
Invoke-Command $MainMenu
$Select = Read-Host
Switch ($Select)
    {
    1 {
       Write-Host
       Write-Host " Select virtual machines to Export."
       Get-VM | Out-GridView -Title "Select virtual machines to export" -PassThru | Export-VM -Path H:\VMExport
       cls
       Write-Host
       Write-Host " Selected virtual machines have been exported."
       cls
       }
    2 {
       Write-Host
       Write-Host " Select virtual machines to to deleted."
       Get-VM | Out-GridView -Title "Select virtual machines to be deleted" -PassThru | Remove-VM -Confirm
       cls
       Write-Host
       Write-Host " Selected virtual machines have been deleted."
       cls
       }
    3 {
       Write-Host
       Write-Host " Select virtual machines to start."
       Get-VM | Out-GridView -Title "Select virtual machines to be started"  -PassThru | Start-VM
       cls
       Write-Host
       Write-Host " Selected virtual machines have been started."
       cls
       }
    4 {
       Write-Host
       Write-Host " Select virtual machines to stop."
       Get-VM | Out-GridView -Title "Select virtual machines to be stopped"  -PassThru | Stop-VM
       cls
       Write-Host
       Write-Host " Selected virtual machines have been stopped."
       cls
       }
    }
}
While ($Select -ne 5)

```

