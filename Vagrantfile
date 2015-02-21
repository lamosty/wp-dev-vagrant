# -*- mode: ruby -*-
# vi: set ft=ruby :

ANSIBLE_PATH = '.' # path targeting Ansible directory (relative to Vagrantfile)

config_file = File.join(ANSIBLE_PATH, 'group_vars/web-dev')

if File.exists?(config_file)
  ansible_configs = YAML.load_file(config_file)

  local_www_root = ansible_configs["local_www_root"]
  remote_www_root = ansible_configs["remote_www_root"]

  local_mysql_data_dir = ansible_configs["local_mysql_data_dir"]
  remote_mysql_data_dir = ansible_configs["remote_mysql_data_dir"]
else
  raise 'group_vars/web-dev file not found. Please set `ANSIBLE_PATH` in Vagrantfile'
end

unless Vagrant.has_plugin?("vagrant-vbguest")
    raise 'vagrant-vbguest is not installed: vagrant plugin install vagrant-vbguest'
end

Vagrant.configure('2') do |config|
  config.vm.box = 'lamosty/web-dev'
  config.vbguest.auto_update = false

  # disable default folder syncing
  config.vm.synced_folder '.', '/vagrant', disabled: true

  # Required for NFS to work, pick any local IP
  config.vm.network :private_network, ip: '192.168.50.5'
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  # for nodejs server
  config.vm.network "forwarded_port", guest: 8090, host: 8090

  # for nodejs livereload
  config.vm.network "forwarded_port", guest: 35729, host: 35729

  config.vm.network "forwarded_port", guest: 8443, host: 8443

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

  # needed for MariaDB service to start after mounting the shared data dir
  config.vm.provision :shell, :inline => "sudo service mysql start", run: 'always'

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
      config.vm.synced_folder local_www_root, remote_www_root, owner: 'www-data', group: 'www-data', mount_options: ['dmode=776', 'fmode=775']
    else

      # Bindfs used for correct permissions

      if !Vagrant.has_plugin? 'vagrant-bindfs'
        raise Vagrant::Errors::VagrantError.new,
          "vagrant-bindfs missing, please install the plugin:\nvagrant plugin install vagrant-bindfs"
      else
       
        config.vm.synced_folder local_www_root, "/vagrant-nfs", type: 'nfs'
        config.vm.synced_folder local_mysql_data_dir, "/vagrant-mysql", type: 'nfs'

        config.bindfs.bind_folder "/vagrant-nfs", remote_www_root, u: 'www-data', g: 'vagrant', p: 'u=rwx:g=rwx:o=rwx', chown_ignore: true, chmod_ignore: true, m: 'vagrant,www-data'

        # Can't execute this command before user 'mysql' is created. It is executed in 'mariadb' ansible role.
        config.bindfs.bind_folder "/vagrant-mysql", remote_mysql_data_dir, u: 'root', g: 'root', multithreaded: true, o: 'nonempty'        
      end
    end
end




