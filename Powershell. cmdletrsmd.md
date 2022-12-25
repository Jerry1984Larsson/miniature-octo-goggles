# Powershell. cmdletrsmd

- **Windows+Tab:** Open Task View
- **Windows+Ctrl+Left or Right Arrow:** Switch between virtual desktops
- **Windows+Ctrl+D:** Create a new Virtual Desktop
- **Arrow Keys and Enter:** Use in Task View to select a Virtual Desktop
- **Delete:** Pressing this key while Task View is open will remove the selected desktop.
- **Escape:** Close Task View

-

## Powershell

### Map Network Drives

```powershell
New-PSDrive –Name “K” –PSProvider FileSystem –Root “\\touchsmart\share” –Persist
```
