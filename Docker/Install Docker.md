# Install Docker in Debian 9 servers
## Update System
```
apt-get install -y vim mc uml-utilities ntp qemu-guest-agent \
htop sudo curl git-core etckeeper molly-guard apt-transport-https ca-certificates \
bridge-utils gettext-base jq -y < /dev/null
```
### Oh-my-zsh

```
apt-get install zsh -y < /dev/null
usermod root -s /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
sed -ri 's/ZSH_THEME="robbyrussell"/ZSH_THEME="agnoster"/g' .zshrc
sed -ri 's/plugins=\(git\)/plugins=\(debian apt systemd docker zsh-navigation-tools\)/g' .zshrc
echo 'export VTYSH_PAGER=more' >> /etc/zsh/zshenv
source .zshrc
```

### Repo y kernel de backports

```
echo "deb http://ftp.debian.org/debian jessie-backports main"  >> /etc/apt/sources.list.d/debian-backports.list
apt update
apt install linux-image-amd64 linux-headers- -t jessie-backports -y < /dev/null
```

### Quitamos Kernel viejo

```
apt remove --purge $(dpkg --list | grep linux-image-3 | cut -d " " -f 3) -y
shutdown -r now
```

### Tunning sistema

```
cat > /etc/sysctl.d/local.conf << EOL
fs.file-max = 2097152
fs.nr_open = 2097152
net.ipv4.ip_nonlocal_bind = 1
EOL

cat > /etc/security/limits.d/local.conf <<EOL
*         hard    nofile      999999
root      hard    nofile      999999
*         soft    nofile      999999
root      soft    nofile      999999
EOL

```

## Install Docker
### Sysdig para monitorización

```
curl -s https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | apt-key add -
curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list
apt-get -qq update < /dev/null
apt-get -qq -y install linux-headers-$(uname -r) < /dev/null || kernel_warning
apt-get -qq -y install sysdig < /dev/null
echo "sysdig-probe" >> /etc/modules-load.d/modules.conf
modprobe sysdig-probe
```

### Instalamos el engine

```
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo "deb https://apt.dockerproject.org/repo debian-jessie main" > /etc/apt/sources.list.d/docker.list
apt-get update && apt-get install -y docker-engine
```

### Cluster docker

-- Si tenemos varios nodos, creamos cluster.

#### En master01
#### Creamos el cluster en master01

```
docker swarm init --advertise-addr IP.ADDR.OF.MASTER01
docker swarm  join-token -q  manager
```
#### Anotamos token de manager

```
	Swarm initialized: current node (q5lwwzmgiysf6p7710gdlv09o) is now a manager.
	To add a worker to this swarm, run the following command:

	    docker swarm join \
	    --token SWMTKN-1-supertoken-of-cluster-swarm \
	    IP.ADDR.OF.MASTER01:2377

	To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

#### Unimos como managers el resto de nodos

```
docker swarm join \
    --token $TOKEN_DE_MANAGER! \
    IP.ADDR.OF.MASTER01:2377
```

#### Preparamos el kernel

```
sed -ri s/quiet/quiet\ cgroup_enable\=memory\ swapaccount\=1/g /etc/default/grub
update-grub
reboot
```
