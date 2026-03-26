# Network related information

## Disable MAC randomisation

As part of securing of internal network, all devices must be registered,
as only registered ones will have access to internal network.


### Windows

To disable MAC address randomization in Windows 10/11:  

1. go to Settings > Network & Internet > Wi-Fi  
2. click Properties  
3. toggle `Random hardware addresses` (or `Use random hardware addresses for this network`) to Off.  

To disable it for a specific network:  

1. go to Settings > Network & Internet > Wi-Fi > Manage known networks  
2. select the network  
3. click Properties  
4. turn off `Random hardware addresses` (or `Use random hardware addresses for this network`).  


!!! Info "Resources"
    * [How do I disable Random MAC Addresses for Ethernet Adapters](https://learn.microsoft.com/en-us/answers/questions/5609414/how-do-i-disable-random-mac-addresses-for-ethernet)


### IOS

Randomized MAC address is configured for each network:

1. Open Settings --> Wi-Fi
2. Tap the (i) icon next to the target network name
3. Locate the `Private Wi-Fi Address` toggle
4. Set it to Off
5. The device will prompt that it will reconnect using the hardware MAC --> confirm


### Android

Randomized MAC address is configured for each network:

1. Open Settings app
2. Select Connections
3. Select Wi-Fi
4. Select ... (the three dots menu)
5. Select `Manage networks`
6. Select target network
6. Select `View more`
7. Select `MAC address type` and chose `Phone MAC`



### Linux


#### NetworkManager


##### Global disable

Create or edit `/etc/NetworkManager/NetworkManager.conf` and add:
```
[device]
wifi.scan-rand-mac-address=no

[connection]
wifi.cloned-mac-address=permanent
ethernet.cloned-mac-address=permanent
```


##### Per connection disable

1. List connections with `nmcli connection show`
2. `nmcli connection modify "YourConnectionName" wifi.cloned-mac-address permanent`
3. `nmcli connection modify "YourConnectionName" 802-3-ethernet.cloned-mac-address permanent`


Then apply changes:

* systemctl reload NetworkManager  

or for NetworkManager.conf changes:  

* nmcli general reload


#### SystemD - Networkd

1. Edit the `.network` file under `/etc/systemd/network/`
2. Add to (or create) section `[Link]`
3. add the option: `MACAddressPolicy=none`

Apply changes:  
`networkctl reload`  
or  
`systemctl restart systemd-networkd`  

