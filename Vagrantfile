Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu/bionic64'
  ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
  config.vm.provider "virtualbox" do |vb| 
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    vb.memory = 2048
    vb.cpus = 2
  end

  config.vm.define 'master' do |master|
    master.vm.hostname = 'master'
    master.vm.provision :shell, path: 'configure-master'
    master.vm.network "private_network", ip: '172.16.1.100'
    # Copy the manifest files over.
    master.vm.provision "file", source: "kube-flannel.yml", destination: "/home/vagrant/kube-flannel.yml"
    master.vm.provision "file", source: "deploys.yml", destination: "/home/vagrant/deploys.yml"
    # Run CNI flannel manifest file.
    master.vm.provision :shell, inline: 'kubectl apply -f /home/vagrant/kube-flannel.yml', run:"always", privileged: false
    # Run manifest file with 2 ub containers.
    master.vm.provision :shell, inline: 'kubectl create -f /home/vagrant/deploys.yml', run:"always", privileged: false
  end

  config.vm.define 'worker01' do |worker|
    worker.vm.provision :shell, path: 'configure-worker01'
    worker.vm.hostname = 'worker01'
    worker.vm.network "private_network", ip: '172.16.1.101'
  end

  config.vm.define 'worker02' do |worker|
    worker.vm.provision :shell, path: 'configure-worker02'
    worker.vm.hostname = 'worker02'
    worker.vm.network "private_network", ip: '172.16.1.102'
  end

end
