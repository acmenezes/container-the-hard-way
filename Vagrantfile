Vagrant.configure("2") do |config|
		config.vm.provider "libvirt" do |v|
			v.memory = 4096
			v.cpus = 2
			v.storage :file, :size => '1G', :type => 'raw'
		end
		config.vm.hostname = "container-host"
		config.vm.box = "centos/7"
end
