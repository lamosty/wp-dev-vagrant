# -*- mode: ruby -*-
# vi: set ft=ruby :

ANSIBLE_PATH = '.' # path targeting Ansible directory (relative to Vagrantfile)

Vagrant.configure('2') do |config|
  config.vm.box = 'lamosty/web-dev'

  # Required for NFS to work, pick any local IP
  config.vm.network :private_network, ip: '192.168.50.5'

  # Send our public ssh keys to the VM
  config.ssh.forward_agent = true

  config.vm.provision :ansible do |ansible|
    ansible.playbook = File.join(ANSIBLE_PATH, 'playbook.yml')

    # 'default' is the alias Vagrant generates for Ansible provisioner. Our host is called 'web-dev', so we need to pair it.
     ansible.groups = {
      'web-dev' => ['default']
    }

    ansible.extra_vars = {
      ansible_ssh_user: 'vagrant',
      user: 'vagrant'
    }

  end

  config.vm.provider 'virtualbox' do |vb|
    # Give VM access to all cpu cores on the host
    cpus = case RbConfig::CONFIG['host_os']
      when /darwin/ then `sysctl -n hw.ncpu`.to_i
      when /linux/ then `nproc`.to_i
      else 2
    end

    # Customize memory in MB
    vb.customize ['modifyvm', :id, '--memory', 2048]
    vb.customize ['modifyvm', :id, '--cpus', cpus]

    # Fix for slow external network connections
    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end
end

