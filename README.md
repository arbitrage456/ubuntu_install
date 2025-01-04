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

pool 4개인가 있는데 맨위에 한놈 살리고 그 위에 아래 내용 넣기 (공백 없어야됨)
```bash
# 시간 동기화 서버 설정
server time2.kriss.re.kr iburst
```

```bash
sudo systemctl restart chrony

chronyc tracking
sudo timedatectl set-ntp true
sudo timedatectl set-timezone Asia/Seoul


sudo systemctl enable chrony
sudo systemctl status chrony

timedatectl show | grep NTP

```

3. nfs 설정

```bash
sudo apt update
sudo apt install -y nfs-common

sudo mkdir -p /mnt/nfs
sudo mount 192.168.88.249:/volume1/arbitrage /mnt/nfs

sudo nano /etc/fstab
```

```bash
192.168.88.249:/volume1/arbitrage /mnt/nfs nfs rw,vers=4,auto,_netdev 0 0
```

```bash
sudo mount -a
```


4. nas 화이트 리스트 등록

```bash
all user admin?
```

5. nodejs 설치

```bash
sudo apt update
sudo apt install -y nodejs npm
node -v
npm -v

```

6. githumb 설치 (필요 시)
```bash
sudo apt update
sudo apt install git -y
git --version

git config --global user.name don3030
git config --global user.email shycrystal@naver.com
git config --list
```

보안 등록
```bash
ssh-keygen -t ed25519 -C shycrystal@naver.com
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

출력된거 깃헙 ssh로 등록
```bash
cd ~/nodejs/Data_Save
git remote set-url origin git@github.com:arbitrage456/Data_Save.git
ssh -T git@github.com
git pull origin main

```

