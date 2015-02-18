# -*- mode: ruby -*-
# vi: set ft=ruby :

ANSIBLE_PATH = '.' # path targeting Ansible directory (relative to Vagrantfile)

config_file = File.join(ANSIBLE_PATH, 'group_vars/web-dev')

if File.exists?(config_file)
  ansible_configs = YAML.load_file(config_file)

  local_www_root = ansible_configs["local_www_root"]
  remote_www_root = ansible_configs["remote_www_root"]

else
  raise 'group_vars/web-dev file not found. Please set `ANSIBLE_PATH` in Vagrantfile'
end


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

  # Configure NFS

  if Vagrant::Util::Platform.windows?
    wordpress_sites.each do |site|
      config.vm.synced_folder local_www_root, remote_www_root, owner: 'vagrant', group: 'www-data', mount_options: ['dmode=776', 'fmode=775']
    end
  else

    # Bindfs used for correct permissions

    if !Vagrant.has_plugin? 'vagrant-bindfs'
      raise Vagrant::Errors::VagrantError.new,
        "vagrant-bindfs missing, please install the plugin:\nvagrant plugin install vagrant-bindfs"
    else
      wordpress_sites.each do |site|
        config.vm.synced_folder local_www_root, "/vagrant-nfs", type: 'nfs'
        config.bindfs.bind_folder "/vagrant-nfs", remote_www_root, u: 'vagrant', g: 'www-data'
      end
    end
  end
end



