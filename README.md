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
sudo apt update
sudo apt install chrony
sudo nano /etc/chrony/chrony.conf
```

pool 4개인가 있는데 맨위에 한놈 살리고 그 위에 아래 내용 넣기
```bash
# 시간 동기화 서버 설정
server time2.kriss.re.kr iburst

# 동기화 빈도 설정
driftfile /var/lib/chrony/chrony.drift      # 시간 오차 데이터를 저장
rtcsync                                   # 시스템 부팅 시 RTC와 동기화
makestep 1.0 3                            # 오차가 1초 이상이면 초기 3회 빠르게 조정

```

```bash
sudo systemctl restart chrony

chronyc tracking
sudo timedatectl set-ntp true
sudo timedatectl set-timezone Asia/Seoul


chronyc sources -v

sudo systemctl enable chrony
sudo systemctl status chrony

timedatectl show | grep NTP

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

