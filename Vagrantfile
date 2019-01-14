# -*- mode: ruby -*-
# vi: set ft=ruby :

# Loading wifi credentials
require 'yaml'
wificred = YAML::load(File.open('wifi-credentials.yml'))

Vagrant.configure("2") do |config|
  config.vm.provision "shell", run: "always", inline: <<-SHELL
      # kernel version to compile and install
      if [ -f /etc/redhat-release ]; then
        # export KERNEL_VERSION=3.16.57
        export KERNEL_VERSION=4.19.13
        export KERNEL_MAJORRELEASE=$(echo $KERNEL_VERSION | grep "^[3-4]" -o)
        export RUNNING_KERNEL=$(uname -r)
        export GITREPO=https://github.com/likwidoxigen/mt7610u_wifi
      fi
      if [ -f /etc/debian_version ]; then
        export KERNEL_VERSION=4.19.13
        export KERNEL_MAJORRELEASE=$(echo $KERNEL_VERSION | grep "^[3-4]" -o)
        export RUNNING_KERNEL=$(uname -r)
        export GITREPO=https://github.com/xtknight/mt7610u-linksys-ae6000-wifi-fixes
      fi
      # getting personal configuration
      export COMPILE_KERNEL=no
      export MAC_ADDRESS=#{wificred['MAC_ADDRESS']}
      export MYSSID=#{wificred['MYSSID']}
      export MYPASSPHRASE=#{wificred['MYPASSPHRASE']}
      export GWIP=#{wificred['MYSSID']}

      if [ "$RUNNING_KERNEL" != "$KERNEL_VERSION" ]  && [ "$COMPILE_KERNEL" = "yes" ]; then
          if [ -f /etc/debian_version ]; then
              apt-get update
              apt-get install build-essential git wireless-tools libx32ncurses5-dev wpasupplicant libelf-dev libssl-dev flex bison -y
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

          if [ "$COMPILE_KERNEL" = "yes" ]; then
            cd /usr/src/
            wget https://cdn.kernel.org/pub/linux/kernel/v$KERNEL_MAJORRELEASE.x/linux-$KERNEL_VERSION.tar.xz
            tar xvpf linux-$KERNEL_VERSION.tar.xz
            cd linux-$KERNEL_VERSION/

            if [ -f /etc/redhat-release ]; then
                cp -a /vagrant/files/config-$KERNEL_VERSION-centos /usr/src/linux-$KERNEL_VERSION/.config
            fi
            if [ -f /etc/debian_version ]; then
                cp -a /vagrant/files/config-$KERNEL_VERSION-ubuntu /usr/src/linux-$KERNEL_VERSION/.config
                ln -s /usr/src/linux-$KERNEL_VERSION /usr/src/linux-headers-$KERNEL_VERSION
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
      fi

      if [ -f /etc/debian_version ] && [ "$COMPILE_KERNEL" = "no" ] && [ ! "$RUNNING_KERNEL" = "4.19.15-041915-generic" ]; then
        cd /usr/local/src
        wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.19.15/linux-image-unsigned-4.19.15-041915-generic_4.19.15-041915.201901130432_amd64.deb
        wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.19.15/linux-headers-4.19.15-041915-generic_4.19.15-041915.201901130432_amd64.deb
        wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.19.15/linux-headers-4.19.15-041915_4.19.15-041915.201901130432_all.deb
        wget https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.19.15/linux-modules-4.19.15-041915-generic_4.19.15-041915.201901130432_amd64.deb
        dpkg -i linux-modules-4.19.15-041915-generic_4.19.15-041915.201901130432_amd64.deb
        dpkg -i linux-image-unsigned-4.19.15-041915-generic_4.19.15-041915.201901130432_amd64.deb
        dpkg -i linux-headers-4.19.15-041915_4.19.15-041915.201901130432_all.deb
        dpkg -i linux-headers-4.19.15-041915-generic_4.19.15-041915.201901130432_amd64.deb
        mkdir -p /lib/firmware/mediatek/
        cp -a /vagrant/files/mediatek-firmware/* /lib/firmware/mediatek/
        # shutdown -h now
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
          # cp -a /vagrant/files/ifcfg-ra0-ubuntu /etc/network/interfaces.d/ra0.cfg
          # ifconfig ra0 up
          # echo "1" > /proc/sys/net/ipv4/ip_forward && iptables -t nat -A POSTROUTING -o ra0 -j MASQUERADE
          # route -n | grep 10.0.2.2 -o 2&>1 /dev/null
          # VB_DEFAULT_ROUTE=$?
          # if [ $VB_DEFAULT_ROUTE -eq 2 ]; then
          #     route del default gw 10.0.2.2
          # fi
          # route -n | grep $GWIP -o 2&>1 /dev/null
          # WIFI_DEFAULT_ROUTE=$?
          # if [ $WIFI_DEFAULT_ROUTE -eq 2 ];then
          #     route add default gw $GWIP
          # fi
          apt-get autoremove -y
      fi

  SHELL

  config.vm.define "ubuntu" do |ubuntu|
      ubuntu.vm.hostname = "ubuntu-wifi"
      ubuntu.vm.box = "bento/ubuntu-18.04"
      # ubuntu.vm.box_check_update = false
      ubuntu.vm.network "private_network", ip: "192.168.2.10"
      # config.vm.network "public_network"
      # ubuntu.ssh.username = 'vagrant'
      # ubuntu.ssh.password = 'vagrant'
      # ubuntu.vbguest.auto_update = false
      ubuntu.vm.boot_timeout = 300

      config.vm.provider "virtualbox" do |vb|
          vb.name = "ubuntu-wifi"
          vb.memory = "1000"
          # vb.customize ["modifyvm", :id, "--usb", "on"]
          # vb.customize ["modifyvm", :id, "--usbxhci", "on"]
          #vendorid and productid from vboxmanage list usbhost
          # vb.customize ['usbfilter', 'add', '0', '--target', :id, '--name', 'ESP', '--vendorid', '0x0b05', '--productid', '0x17d1']
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
          vb.name = "centos-wifi"
          vb.memory = "512"
          vb.customize ["modifyvm", :id, "--usb", "on"]
          vb.customize ["modifyvm", :id, "--usbxhci", "on"]
          #vendorid and productid from vboxmanage list usbhost
          vb.customize ['usbfilter', 'add', '0', '--target', :id, '--name', 'ESP', '--vendorid', '0x0b05', '--productid', '0x17d1']
      end
  end
end
