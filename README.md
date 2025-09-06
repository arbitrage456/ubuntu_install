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
  bridges:
    br0:
      dhcp4: true
      dhcp6: false
      interfaces:
        - vlan50   #스위칭허브 id
  vlans:
    vlan50:
      dhcp4: false
      dhcp6: false
      id: 50
      link: "enp42s0"
```

```bash
sudo netplan apply
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

쿠버네티스 설치 (설치만 해당, 방화벽 설정, 워커노드, 마스터노드 설정은 개별)
```
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Update and upgrade packages
sudo apt update
sudo apt upgrade -y

# Disable swap for better performance
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load kernel modules for containerd and Kubernetes
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl for Kubernetes networking
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl settings
sudo sysctl --system

# Install required tools and CA certificates
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

# Add Docker repository and install containerd
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y containerd.io

# Configure containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Add Kubernetes repository
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes components
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Enable Kubernetes components to start on boot
sudo systemctl enable kubelet

# Automatically handle deferred service restarts
sudo needrestart -r a || true
```


마스터노드 설정할 때만 해당 (마스터 노드 네트워크 설정)
```
# Print completion message
echo "Kubernetes setup completed successfully!"

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chmod 644 /etc/kubernetes/admin.conf

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```





```
sudo apt update
sudo apt install openssh-server

sudo systemctl enable ssh
sudo systemctl start ssh

sudo nano /etc/ssh/sshd_config
접근해서 아래 내용 주석해제
PasswordAuthentication yes

sudo systemctl restart ssh
sudo ufw allow ssh
sudo ufw enable
```

vscode 플러그인 설치  
Remote - SSH  
컨트롤 + shift + p -> Tunnels/SSH  
Add New ssh host  
ssh {pc유저이름}@{아이피} -A  

### ssh OTP 등록하기
```
sudo apt update && sudo apt install libpam-google-authenticator -y
google-authenticator
```
모두 y 체크 후 엔터하면 QR 코드가 나옴. 그거 스캔

```
sudo nano /etc/pam.d/sshd

# 아래 줄 추가 아무데나 추가하면 됨.
auth required pam_google_authenticator.so

sudo nano /etc/ssh/sshd_config
# 아래 설정 주석 해제 및 추가
ChallengeResponseAuthentication yes
AuthenticationMethods keyboard-interactive

# 파일 저장 후 ssh 다시 실행
sudo systemctl restart ssh
```






