# caldera-makejail
MITRE Caldera makejail is an [AppJail-makejail](https://github.com/AppJail-makejails) used by deploy a testing [MITRE Caldera](https://caldera.mitre.org/)) a scalable, automated adversary emulation platform . The principal goals are helps us to fast way install, configure and run MITRE Caldera into a FreeBSD jail. It can be helpful to easily automate adversary emulation, assist manual red-teams, and automate incident response. 

![image](https://github.com/alonsobsd/caldera-makejail/assets/11150989/57773d41-b5c4-44e1-8d7d-19927fe67364)

![image](https://github.com/alonsobsd/caldera-makejail/assets/11150989/3ac08704-d88e-47b6-8100-be0cd187a509)

![image](https://github.com/alonsobsd/caldera-makejail/assets/11150989/ca7c162e-94a0-4e9f-884f-5e49d73eebd9)

![image](https://github.com/alonsobsd/caldera-makejail/assets/11150989/15f5dc38-8c4a-4bf6-94ee-43d8d2593ca7)

## Requirements
Before you can install MITRE Caldera using this makejail you need some initial configurations

#### Enable Packet filter
We need add somes lines to /etc/rc.conf

```sh
# sysrc pf_enable="YES"
# sysrc pflog_enable="YES"

# cat << "EOF" >> /etc/pf.conf
nat-anchor 'appjail-nat/jail/*'
nat-anchor "appjail-nat/network/*"
rdr-anchor "appjail-rdr/*"
EOF
# service pf reload
# service pf restart
# service pflog restart
```
rdr-anchor section is necessary for use dynamic redirect from jails

### Enable forwarding
```sh
# sysrc gateway_enable="YES"
# sysctl net.inet.ip.forwarding=1
```
#### Bootstrap a FreeBSD version
Before you can begin creating containers, AppJail needs fetch and extract components for create jails. If you are creating FreeBSD jails it must be a version equal or lesser than your host version. In this example we will create a 13.2-RELEASE bootstrap

```sh
# appjail fetch
```
#### Create a virtualnet
Create a virtualnet for add MITRE Caldera jail from caldera-makejail. Otherwise you can use your own virtualnet if you created it previously

```sh
# appjail network add caldera-net 10.0.0.0/24
```
it will create a bridge named caldera-net in where Caldera jail epair interfaces will be attached. By default caldera-makejail will use NAT for internet outbound. Do not forget added a pass rule to /etc/pf.conf because caldera-makefile will try to download and install packages and some another resources for configuration of it

```sh
pass out quick on caldera-net inet proto { tcp udp } from 10.0.0.2 to any
```

#### Create a lightweight container system
Create a container named caldera with a private IP address 10.0.0.2. Take on mind IP address must be part of caldera-net network

```sh
# appjail makejail -f gh+alonsobsd/caldera-makejail -j caldera -- --network caldera-net --caldera_ip 10.0.0.2
```
When it is done you will see credentials info for connect to MITRE Caldera via web browser.

```sh
 ################################################ 
 MITRE Caldera admin credential                   
 Hostname : https://jail-host-ip:8443             
 Username : admin                                 
 Password : Z1EtVnltRtirHDOTVY4=                          
 ################################################ 
 
 ################################################ 
 MITRE Caldera blue credential                    
 Hostname : https://jail-host-ip:8443             
 Username : blue                                  
 Password : M0WmJnQOLG3va+b0LM8=                           
 ################################################ 
  
 ################################################ 
 MITRE Caldera red credential                     
 Hostname : https://jail-host-ip:8443             
 Username : red                                   
 Password : 1TPza2NLp0h1scaZ2uA=                            
 ################################################
 ```
Keep them to another secure place

## License
This project is licensed under the BSD-3-Clause license.
