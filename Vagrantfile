# -*- mode: ruby -*-
# vi: set ft=ruby :

# Loading wifi credentials
require 'yaml'
wificred = YAML::load(File.open('wifi-credentials.yml'))

Vagrant.configure("2") do |config|
  config.vm.provision "shell", run: "always", inline: <<-SHELL
      # kernel version to compile and install
      export KERNEL_VERSION=3.16.57
      export KERNEL_MAJORRELEASE=$(echo $KERNEL_VERSION | grep "^[3-4]" -o)
      export RUNNING_KERNEL=$(uname -r)
      export GITREPO=https://github.com/likwidoxigen/mt7610u_wifi # 3.16.57
      # getting personal configuration
      export MAC_ADDRESS=#{wificred['MAC_ADDRESS']}
      export MYSSID=#{wificred['MYSSID']}
      export MYPASSPHRASE=#{wificred['MYPASSPHRASE']}
      export GWIP=#{wificred['MYSSID']}

      #########################################

      # export GITREPO=https://github.com/likwidoxigen/mt7610u_wifi # 3.16.57
          # CentOS 7 - stable
          # Ubuntu 14.04 LTS - unstable
          # not working - ifconfig ra0 up - Operation not supported
          # should work on debian/ubuntu
      # export GITREPO=https://github.com/xtknight/mt7610u-linksys-ae6000-wifi-fixes
          # 4.7 - not working - ifconfig ra0 - Operation not supported
      # export GITREPO=https://github.com/ulli-kroll/mt7610u # 4.15.6 -
          # insmod: ERROR: could not insert module mt7610u.ko: Unknown symbol in module
      # export GITREPO=https://github.com/regalstreak/mt7610u_wifi_sta_v3002_dpo_20130916 # 4.1.7 - raspberry-pi
          # ifconfig ra0
          # ra0: error fetching interface information: Device not found
      # export GITREPO=https://github.com/chenhaiq/mt7610u_wifi_sta_v3002_dpo_20130916 # - 4.1.7-v7+ armv7l GNU/Linux
          # ifconfig ra0
          # ra0: error fetching interface information: Device not found
      # export GITREPO=https://github.com/rebrane/mt7610u # 4.14.1 or 4.15.0
          # failt to compile on 4.14.1
      # export GITREPO=https://github.com/CooperCao/MT7610U_Kali_Linux_64Bit # 4.14.0
          # ifconfig ra0
          # ra0: error fetching interface information: Device not found
      # export GITREPO=https://github.com/benperove/mt7610u_wifi # 4.14.37 + ARM - make -j4, make install, cp RT2870STA.dat /etc/Wireless/RT2870STA/RT2870STA.dat
          # ifconfig ra0
          # ra0: error fetching interface information: Device not found
      # export GITREPO=https://github.com/Hygens/mt7610u_wifi_sta_v3002_dpo_20130916 # 4.9.2
          # fail to compile
      # export GITREPO=https://github.com/Myria-de/mt7610u_wifi_sta_v3002_dpo_20130916 - 4.9.2
          # fail to compile
      #export GITREPO=https://bitbucket.org/Tijee/mt7610u.git
          #- Ubuntu 14.04 LTS 64bit - 3.13.11
      #export GITREPO=https://bitbucket.org/sanrath/mediatek_mt7610u_sta_driver_linux-64bit.git
          # - Ubuntu 14.04 LTS 64bit - 3.13.11

      if [ "$RUNNING_KERNEL" != "$KERNEL_VERSION" ]; then
          if [ -f /etc/debian_version ]; then
              apt-get update
              apt-get install build-essential git wireless-tools libx32ncurses5-dev wpasupplicant -y
              apt-get upgrade -y
          fi
          if [ -f /etc/redhat-release ]; then
              yum upgrade -y
              yum groupinstall 'Development tools'  -y
              yum install kernel-devel unzip usbutils epel-release wget -y
              yum install wget redhat-lsb-core openssl-devel libxml2-devel libxslt-devel gd-devel perl-ExtUtils-Embed GeoIP-devel pcre-devel unzip which -y
              yum install ncurses-devel qt-devel hmaccalc zlib-devel binutils-devel elfutils-libelf-devel -y
              yum install wireless-tools -y
          fi

          cd /usr/src/
          wget https://cdn.kernel.org/pub/linux/kernel/v$KERNEL_MAJORRELEASE.x/linux-$KERNEL_VERSION.tar.xz
          tar xvpf linux-$KERNEL_VERSION.tar.xz
          # ln -s linux-$KERNEL_VERSION linux-headers-$KERNEL_VERSION
          cd linux-$KERNEL_VERSION/

          if [ -f /etc/redhat-release ]; then
              cp -a /vagrant/files/config-$KERNEL_VERSION-centos /usr/src/linux-$KERNEL_VERSION/.config
          fi
          if [ -f /etc/debian_version ]; then
              cp -a /vagrant/files/config-$KERNEL_VERSION-ubuntu /usr/src/linux-$KERNEL_VERSION/.config
          fi

          # Building new kernel for CentOS or Ubuntu
          make bzImage
          make modules
          make modules_install
          cp /usr/src/linux-$KERNEL_VERSION/arch/x86_64/boot/bzImage /boot/vmlinuz-$KERNEL_VERSION

          # CentOS
          if [ -f /etc/redhat-release ]; then
              cp -afu /usr/src/linux-$KERNEL_VERSION/.config /boot/config-$KERNEL_VERSION
              mkinitrd /boot/initramfs-$KERNEL_VERSION.img $KERNEL_VERSION --force
              grub2-set-default 0
              grub2-mkconfig -o /boot/grub2/grub.cfg
          fi
          # ubuntu
          if [ -f /etc/debian_version ]; then
              cp -a /usr/src/linux-$KERNEL_VERSION/.config /boot/config-$KERNEL_VERSION
              update-initramfs -c -k $KERNEL_VERSION
              sed -i 's/GRUB_HIDDEN_TIMEOUT=0/GRUB_HIDDEN_TIMEOUT=1/g' /etc/default/grub
              sed -i 's/GRUB_HIDDEN_TIMEOUT_QUIET=true/GRUB_HIDDEN_TIMEOUT_QUIET=false/g' /etc/default/grub
              grub-set-default 0
              update-grub
          fi
          shutdown -h now
      fi

      if [ ! -f /usr/src/mt7610u/os/linux/mt7650u_sta.ko ]; then
        rm -rf /usr/src/mt7610u
        cd /usr/src
        git clone $GITREPO /usr/src/mt7610u
        cd /usr/src/mt7610u
        make
        make install
      fi

      # CentOS
      if [ -f /etc/redhat-release ]; then
          cp -a /vagrant/files/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant.conf
          sed -i "s/MYSSID/$MYSSID/g" /etc/wpa_supplicant/wpa_supplicant.conf
          sed -i "s/MYPASSPHRASE/$MYPASSPHRASE/g" /etc/wpa_supplicant/wpa_supplicant.conf
          wpa_passphrase "$MYSSID" $MYPASSPHRASE >> /etc/wpa_supplicant/wpa_supplicant.conf
          cp -a /vagrant/files/ifup-wireless /etc/sysconfig/network-scripts/ifup-wireless
          cp -a /vagrant/files/ifcfg-wlan0 /etc/sysconfig/network-scripts/ifcfg-ra0
          sed -i "s/MAC_ADDRESS/$MAC_ADDRESS/g" /etc/sysconfig/network-scripts/ifcfg-ra0
          sed -i "s/MYSSID/$MYSSID/g" /etc/sysconfig/network-scripts/ifcfg-ra0
          cp -a /vagrant/files/rc.local /etc/rc.d/rc.local
          systemctl disable wpa_supplicant
          systemctl disable NetworkManager
          systemctl stop NetworkManager
          wpa_supplicant -i ra0 -c /etc/wpa_supplicant/wpa_supplicant.conf -B -D wext
          dhclient ra0
          echo "1" > /proc/sys/net/ipv4/ip_forward && iptables -t nat -A POSTROUTING -o ra0 -j MASQUERADE
      fi
      # Ubuntu
      if [ -f /etc/debian_version ]; then
          cp -a /vagrant/files/ifcfg-ra0-ubuntu /etc/network/interfaces.d/ra0.cfg
          ifconfig ra0 up
          echo "1" > /proc/sys/net/ipv4/ip_forward && iptables -t nat -A POSTROUTING -o ra0 -j MASQUERADE
          route -n | grep 10.0.2.2 -o 2&>1 /dev/null
          VB_DEFAULT_ROUTE=$?
          if [ $VB_DEFAULT_ROUTE -eq 2 ]; then
              route del default gw 10.0.2.2
          fi
          route -n | grep $GWIP -o 2&>1 /dev/null
          WIFI_DEFAULT_ROUTE=$?
          if [ $WIFI_DEFAULT_ROUTE -eq 2 ];then
              route add default gw $GWIP
          fi
          apt-get autoremove -y
      fi

      # wget http://dlcdnet.asus.com/pub/ASUS/wireless/USB-AC51/UT_USB_AC51_3001_Linux.zip
      # unzip UT_USB_AC51_3001_Linux.zip
      # cd UT_USB_AC51_3001_Linux
      # tar xvpf mt7610u_wifi_sta_v3001_dpo_20130725.tar
      # cd mt7610u_wifi_sta_v3001_dpo_20130725/
      # make
      # make install
  SHELL

  config.vm.define "ubuntu" do |ubuntu|
      ubuntu.vm.hostname = "ubuntu-wifi"
      ubuntu.vm.box = "ubuntu/trusty64"
      # ubuntu.vm.box_check_update = false
      ubuntu.vm.network "private_network", ip: "192.168.2.10"
      # config.vm.network "public_network"
      # ubuntu.vm.synced_folder "./files", "/vagrant"
      ubuntu.ssh.username = 'vagrant'
      ubuntu.ssh.password = 'vagrant'
      ubuntu.vbguest.auto_update = false

      config.vm.provider "virtualbox" do |vb|
          # #   vb.gui = true
          vb.memory = "1024"
          vb.customize ["modifyvm", :id, "--usb", "on"]
          vb.customize ["modifyvm", :id, "--usbxhci", "on"]
          #vendorid and productid from vboxmanage list usbhost
          vb.customize ['usbfilter', 'add', '0', '--target', :id, '--name', 'ESP', '--vendorid', '0x0b05', '--productid', '0x17d1']
      end
  end

  config.vm.define "centos" do |centos|
      centos.vm.box = "bento/centos-7"
      centos.vm.hostname = "centos-wifi"
      centos.vm.network "private_network", ip: "192.168.2.20"
      # centos.vm.box_check_update = false
      # config.vm.network "public_network"
      # ubuntu.vm.synced_folder "./files", "/vagrant"
      # centos.ssh.username = 'vagrant'
      # centos.ssh.password = 'vagrant'
      # centos.vbguest.auto_update = false

      config.vm.provider "virtualbox" do |vb|
          # #   vb.gui = true
          vb.memory = "1024"
          vb.customize ["modifyvm", :id, "--usb", "on"]
          vb.customize ["modifyvm", :id, "--usbxhci", "on"]
          #vendorid and productid from vboxmanage list usbhost
          vb.customize ['usbfilter', 'add', '0', '--target', :id, '--name', 'ESP', '--vendorid', '0x0b05', '--productid', '0x17d1']
      end
  end
end
