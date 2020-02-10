# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  "otuslinux.vagrant.kernel" => {
        :box_name => "centos/7",
        :cpus => 4,
        :memory => 2048,
        :net => [ip: '192.168.14.149'],
        :forwarded_port => [],
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 1024,
			:port => 1
		}
	}
  },
  "otuslinux.parker.kernel" => {
        :box_name => "centos-7-5",
        :cpus => 4,
        :memory => 2048,
        :net => [ip: '192.168.14.150'],
        :forwarded_port => [],
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 1024,
			:port => 1
		}
	}
  }
}

Vagrant.configure("2") do |config|
        MACHINES.each do |boxname, boxconfig|
                config.vm.synced_folder ".", "/vagrant", disabled: true
                config.vm.define boxname do |box|
                        box.vm.box = boxconfig[:box_name]
                        box.vm.host_name = boxname.to_s

                        #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
                        #box.vm.network "private_network", ip: boxconfig[:ip_addr]

                        # Additional network config if present
                        if boxconfig.key?(:net)
                                boxconfig[:net].each do |ipconf|
                                        box.vm.network "private_network", ipconf
                                end
                        end
                        # Port-forward config if present
                        if boxconfig.key?(:forwarded_port)
                                boxconfig[:forwarded_port].each do |port|
                                        box.vm.network "forwarded_port", port
                                end
                        end

                        if (boxname.to_s === "otuslinux.vagrant.kernel")
                                box.vm.provider :virtualbox do |vb|
                                        vb.memory = boxconfig[:memory]
                                        vb.cpus = boxconfig[:cpus]

                                        needsController = false
                                        boxconfig[:disks].each do |dname, dconf|
                                                unless File.exist?(dconf[:dfile])
                                                vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                                needsController =  true
                                                end

                                        end
                                        if needsController == true
                                                vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                                                boxconfig[:disks].each do |dname, dconf|
                                                        vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                                                end
                                        end
                                end

                        
                                box.vm.provision "file", source: "./manual_kernel_update/packer/scripts/stage-1-kernel-update.sh", destination: "~/kernel-update.sh"
                                box.vm.provision "shell", inline: <<-SHELL
                                        mkdir -p ~root/.ssh
                                        cp ~vagrant/.ssh/auth* ~root/.ssh
                                        echo "nameserver 9.9.9.9" >> /etc/resolv.conf
                                        #yum update -y
                                        yum install -y mdadm smartmontools hdparm gdisk

                                        # kernel update
                                        /bin/sh /home/vagrant/kernel-update.sh

                                        echo `uname -r`
                                SHELL
                        elsif (boxname.to_s === "otuslinux.parker.kernel")
                                box.vm.provision "shell", inline: <<-SHELL
                                        mkdir -p ~root/.ssh
                                        cp ~vagrant/.ssh/auth* ~root/.ssh
                                        echo "nameserver 9.9.9.9" >> /etc/resolv.conf
                                        #yum update -y
                                        yum install -y mdadm smartmontools hdparm gdisk

                                        echo `uname -r`
                                SHELL
                        end
                end
        end
end

