# -*- mode: ruby -*-
# vi: set ft=ruby :
#  _    __                             __  _____ __
# | |  / /___ _____ __________ _____  / /_/ __(_) /__
# | | / / __ `/ __ `/ ___/ __ `/ __ \/ __/ /_/ / / _ \
# | |/ / /_/ / /_/ / /  / /_/ / / / / /_/ __/ / /  __/
# |___/\__,_/\__, /_/   \__,_/_/ /_/\__/_/ /_/_/\___/
#           /____/
#
#
MANAGE_NET = "10.0.0."
#VMTUN_NET = "20.0.0."
#EXTERN_NET = "0"
nodes=[
  {
	:hostname => "controller",
	:ram => 2048,
	:ip_manage => "11",
	:yaml => "controller",  
  },
  {
	:hostname => "compute",
	:ram => 1024 ,
	:ip_manage => "31",
	:ip_vmtunnel => "22",
	:yaml => "compute",
  },
  {
	:hostname => "net",
	:ram => 512 ,
	:ip_manage => "21",
	:ip_extern => "yes",
	:ip_vmtunnel => "32",
	:yaml => "net",  
  }
]

Vagrant.configure(2) do |config|
	# Определяем исходный box 
	config.vm.box = "ubuntu_os"
	# Настройки,необходимые для подключения к libvirt хосту.
	config.vm.provider :libvirt do |libvirt|
           libvirt.driver = "kvm"
           libvirt.host = 'home' 
           libvirt.uri = "qemu+ssh://192.168.9.102/system"
           libvirt.storage_pool_name = 'images'
           libvirt.connect_via_ssh = 'yes'
           libvirt.username ='german'
           libvirt.id_ssh_key_file = '/home/german/.ssh/id_rsa'
           libvirt.disk_bus = "virtio"
           libvirt.nic_model_type = "virtio"
           libvirt.video_type = "qxl"
           libvirt.graphics_type = "spice"
    end
    
    nodes.each do |machine|
        config.vm.synced_folder ".", "/vagrant", disabled: true
        # Применяем конфигурации для каждой машины.Название машины (для гипервизора) находится в переменной "machine[:hostname]"
        config.vm.define machine[:hostname] do |node|
            # Hostname который будет присвоенс ОС (work wth libvirt provider)
            node.vm.hostname = machine[:hostname]
            # Добавление и настройка сетевого адаптера 
            node.vm.network "private_network", ip: MANAGE_NET + machine[:ip_manage]
            # Конфигурация "железа" vm.
            if (!machine[:ip_vmtunnel].nil?)
					node.vm.network "private_network",
						:libvirt__network_name => 'os_vmtunnel',
						:libvirt__dhcp_enabled => "no"
			end
			if (!machine[:ip_extern].nil?)
					node.vm.network "public_network",:dev => "bridge0"
			end
            node.vm.provider :libvirt do |node|
				node.memory = machine[:ram]
				#node.machine_virtual_size ='5G'
				# Дополнительные дисковые устройства для chunk серверов.
				#if (!machine[:hdd].nil?)
				##	node.storage :file , :size => '3G' , :device => "vdh"
			end
		
			node.vm.provision "ansible" do |ansible|
				ansible.verbose = "v"
				ansible.playbook = machine[:yaml] + ".yaml"
				ansible.raw_ssh_args = "-o controlpath=/tmp/ssh%r@%h:%p -o controlmaster=auto -o 'ProxyCommand=ssh -W %h:%p german@192.168.9.102' -o ControlPersist=15m "
			end
		end       
	end
end           
