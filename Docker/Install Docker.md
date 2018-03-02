# Install Docker in two servers
## Update & Upgrade
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

## Install software
### Docker

## sysdig para monitorización

```
curl -s https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | apt-key add -
curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list
apt-get -qq update < /dev/null
apt-get -qq -y install linux-headers-$(uname -r) < /dev/null || kernel_warning
apt-get -qq -y install sysdig < /dev/null
echo "sysdig-probe" >> /etc/modules-load.d/modules.conf
modprobe sysdig-probe
```

## Instalamos en engine

```
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo "deb https://apt.dockerproject.org/repo debian-jessie main" > /etc/apt/sources.list.d/docker.list
apt-get update && apt-get install -y docker-engine
```
