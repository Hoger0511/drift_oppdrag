# Sette opp server
```
laste ned server22 --> velge desktop experience --> velge custom --> laste ned også ta ut usb-en. 
```

# Installere alt, og sette opp statisk ip med Powershell
```
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
 
Install-WindowsFeature -Name DNS -IncludeManagementTools
 
Install-WindowsFeature -Name DHCP -IncludeManagementTools
 
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools
 
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.100 -PrefixLength 24 -DefaultGateway 192.168.1.1
```

