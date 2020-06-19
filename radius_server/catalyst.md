## Catalyst

### Login

```
>enable
#configure terminal
```

### Password

`password`


### Config

`#sh ru`

###

- Hostname

`#hostname SW_NAME`

- Timezone
  `#ntp server`

### Interface

- Single

`#int gi 0/1`
`#no shutdown`

- Multiple
  `#interface range gi 0/1-8`
  `#interface range gi 0/1-4, gi 0/5-8`

### VLAN

- Create vlan
  `#vlan 10`
  `#name vlan10`

- Swithport(access)
  `# switchport`
  `# switchport mode access` #access, trunk, dynamic auto(access), dynamic desirable(trunk)
  `#switchport access vlan 10`

- Switchport(trunk)
  `#switchport trunk encapsulation dot1q`
  `#switchport trunk allowed vlan add 10` #add/remove
  `#switchport trunk allowed vlan add 10-20`

- Switchport(Native vlan) #no tag vlan
  `#switchport trunk native vlan 10`

- Set IPaddress(interface)
  `#interface gi 0/1`
  `#ip address 172.16.0.1 255.255.255.0`

- Set IPaddress(vlan interface)
  `#inteface vlan 10`
  `#ip address 172.16.1.1`

### Link Aggregaton

- LACP
  `#int range ge 0/1-2`
  `#switchport trunk encaplulation dot1q`
  `#switchport mode trunk`
  `#channel-group 1 mode active` #on => static, active => dynamic
  `#int port-channel 1`
  `#switchport access vlan 10`
  `#port-chammel load-balance src-ip dst-ip`
  **note:** max 8 channels

## Radius

- Enable AAA
  `#aaa new-model`

- Register radius server
  `#radius server config-name`
  `#address ipv4 172.16.0.254 auth-port 1812 acct-port 1812` #old standard port is 1645
  `#key XXXXXXXXX`

- Add radius server to authentication server group
  `#aaa group server radius radius-group`
  `#server name radius-grroup`

- s
  `#aaa authentication dot1x default group radius`
  `#dot1x system-auth-control`
  `#aaa authorization network default group radius`
