# PowerShell is a Query Language for JSON

PowerShell, as is, has all the tools you need to query JSON and then reshape, re-purpose it to what you need.

```powershell
$json = @"
{
  "locations": [
    {"name": "Seattle", "state": "WA"},
    {"name": "New York", "state": "NY"},
    {"name": "Bellevue", "state": "WA"},
    {"name": "Olympia", "state": "WA"}
  ]
}
"@ | ConvertFrom-Json

$names=(($json.locations | ? state -eq 'wa').name | Sort) -join ','

@{WashingtonCities = $names} | ConvertTo-Json
```

## Result

```json
{
  "WashingtonCities": "Bellevue,Olympia,Seattle"
}
```
