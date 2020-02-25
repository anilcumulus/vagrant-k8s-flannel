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
    config.vm.network "forwarded_port",
        guest: 8001,
        host: 8001,
        auto_correct: true
    # Copy the manifest files over.
    master.vm.provision "file", source: "kube-flannel.yml", destination: "/home/vagrant/kube-flannel.yml"
    master.vm.provision "file", source: "deploys.yml", destination: "/home/vagrant/deploys.yml"
    master.vm.provision "file", source: "kubernetes-dashboard.yml", destination: "/home/vagrant/kubernetes-dashboard.yml"

    # Run CNI flannel manifest file.
    master.vm.provision :shell, inline: 'kubectl apply -f /home/vagrant/kube-flannel.yml', run:"always", privileged: false
    # Run manifest file with 2 ub containers.
    master.vm.provision :shell, inline: 'kubectl create -f /home/vagrant/deploys.yml', run:"always", privileged: false

    # Create dashboard and show the token for UI login
    master.vm.provision :shell, inline: 'kubectl create -f /home/vagrant/kubernetes-dashboard.yml', run:"always", privileged: false
    $message = <<-MSG
    echo "

Login to the Kubernetes dashboard at: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

Use the token below:

" >> /home/vagrant/token
MSG
    master.vm.provision "shell", inline: $message, run: "always", privileged: false
    master.vm.provision :shell, inline: 'kubectl -n kube-system describe $(kubectl -n kube-system get secret  -oname | grep admin) | grep token: | awk \'{print $2}\' >> /home/vagrant/token', run:"always", privileged: false
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

    worker.trigger.after :up do |trigger|
       trigger.run = {
          inline: "ssh -i .vagrant/machines/master/virtualbox/private_key -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no vagrant@172.16.1.100 cat /home/vagrant/token && screen -d -m kubectl proxy --address=0.0.0.0"
       }
    end
  end

end
