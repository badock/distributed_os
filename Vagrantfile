nodes = [
  { :hostname => 'compute1', :role => 'compute',  :ip => '192.168.0.43', :mac => "5CA1AB1E0001", :box => 'bento/ubuntu-18.04', :ram => 2048 },
  { :hostname => 'control1', :role => 'controller', :ip => '192.168.0.42', :mac => "5CA1AB1E0002", :box => 'bento/ubuntu-18.04', :ram => 2048 }
]

Vagrant.configure("2") do |config|
  VAGRANT_VM_PROVIDER = "virtualbox"
  ANSIBLE_RAW_SSH_ARGS = []

  nodes.each do |node_config|
    config.vm.define node_config[:hostname] do |machine|
      machine.vm.box = "bento/ubuntu-18.04"
      machine.vm.hostname = node_config[:hostname]
      machine.vm.network "private_network", ip: node_config[:ip], mac: node_config[:mac]

      machine.vm.base_mac = nil

      machine.vm.provider "virtualbox" do |machine_virtualbox|
        machine_virtualbox.memory = node_config[:ram]
      end

      machine.vm.provision "shell", inline: "whoami"
      machine.vm.provision "shell", inline: <<-SCRIPT
if [ ! -f /vagrant/sshkeys/id_rsa ];
then
    mkdir -p /vagrant/sshkeys/;
    ssh-keygen -t rsa -N '' -f /vagrant/sshkeys/id_rsa;
fi
      SCRIPT

      machine.vm.provision "shell", inline: "mkdir -p /root/.ssh/"

      if node_config[:role] == "controller"
        machine.vm.provision "shell", inline: "cp /vagrant/sshkeys/id_rsa* ~vagrant/.ssh/."
        machine.vm.provision "shell", inline: "cp /vagrant/sshkeys/id_rsa* /root/.ssh/."

        machine.vm.provision "shell", inline: "chmod 644 ~vagrant/.ssh/id_rsa"
        machine.vm.provision "shell", inline: "chmod 600 ~root/.ssh/id_rsa"
      end

      machine.vm.provision "shell", inline: "cat /vagrant/sshkeys/id_rsa.pub >> ~vagrant/.ssh/authorized_keys"
      machine.vm.provision "shell", inline: "cat /vagrant/sshkeys/id_rsa.pub >> /root/.ssh/authorized_keys"

      # Generate hosts file
      machine.vm.provision "shell", inline: "echo '' > /etc/hosts"
      nodes.each do |node_config2|
        machine.vm.provision "shell", inline: "echo '#{ node_config2[:ip] } #{ node_config2[:hostname] }' >> /etc/hosts"
      end

      # Add swap

      machine.vm.provision "shell", inline: <<-SCRIPT
if [ ! -f /root/configured_swap ];
then
    fallocate -l 4G /swapfile && chmod 0600 /swapfile && mkswap /swapfile && swapon /swapfile && echo '/swapfile none swap sw 0 0' >> /etc/fstab
    echo vm.swappiness = 10 >> /etc/sysctl.conf && echo vm.vfs_cache_pressure = 50 >> /etc/sysctl.conf && sysctl -p
    touch /root/configured_swap
fi
      SCRIPT

      # Only execute once the Ansible provisioner,
      # when all the machines are up and ready.
      if node_config[:hostname] == nodes[-1][:hostname]
        machine.vm.provision "ansible_local" do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          ansible.playbook = "ansible_playbook.yml"

          controllers = nodes.select{|n| n[:role] == "controller" }.map{ |n| n[:hostname]}
          computes = nodes.select{|n| n[:role] == "compute" }.map{ |n| n[:hostname]}

          ansible.groups = {
            "controllers" => controllers,
            "computes" => computes
          }
          ansible.config_file = "ansible.cfg"
          ansible.verbose = "-v"
        end
      end
    end
  end
end