# ubuntu_install

1. 인터넷 설정

```bash
sudo apt update  
sudo apt install isc-dhcp-client  
sudo nano /etc/netplan/01-netcfg.yaml  
```


```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    enp42s0:   #인터넷 포트
      dhcp4: false
      dhcp6: false
    enp4s0f0np0: #내부망 포트
      addresses:
        - 192.168.88.xxx/24  #101=A, 104=D
      dhcp4: false
      dhcp6: false
  brides:
    br0:
      dhcp4: true
      dhcp6: false
      - vlan50   #스위칭허브 id
  vlans:
    vlan50:
      dhcp4: false
      dhcp6: false
      id: 50
      link: "enp42s0"
```

2. 동기화 설정

```bash

```

3. nfs 설정

```bash
```

4. nas 화이트 리스트 등록

```bash
all user admin?
```

5. nodejs 설치

```bash

```

