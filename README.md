# WordPress Development Vagrant Configs

I use these vagrant server configurations for local development of web applications, primarily WordPress related development. Ansible is used for provisioning of newly created server and [vagrant-bindfs](gael-ian/vagrant-bindfs) for proper permissions of NFS shared folders.

## Features

### Ansible playbooks for:
- common operations (setting locales, updating *apt*, installing system packages, copying config files from *host OS* home directory)
- Nginx with reasonable configs and predefined vhosts
- [HHVM](http://hhvm.com/) for extra PHP performance and testing with nice configurations
- MariaDB 10.0 instead of MySQL with shared data between host OS and guest OS
- Composer with HHVM execution (blazingly fast)
- nodejs and NPM for client-side dependencies and development

### Port-forwarding out of the box

Nginx and nodejs ports 8080 and 8090 respectively are forwarded to the host OS upon vagrant up. SSH keys are also forwarded from host OS to the guest OS.

### Fast NFS shared files with fully-working permissions and ownership

Using vagrant-bindfs, you can share codebases between host OS and guest OS with fast NFS support and any chmod/chown of your choosing. Using NPM package manager from within virtual server working too (chmod and chown are ignored).

### Database data sharing (MySQL/MariaDB)

Sharing files is not enough — we also need the databases. Setting MariaDB data dir to a shared location does the trick. However, we need to pay attention to start MariaDB only after data dir has been mounted and shared on the guest OS. All done automatically in the Vagrantfile.

If your host OS MySQL/MariaDB doesn't have *debian-sys-maint* user (mine on OS X doesn't), it will be copied and inserted into the shared database on the first and subsequent virtual server starts. 

## Installation

1. clone the repo
2. change global configurations (group_vars/, roles/*/defaults) to match your needs/host OS paths
3. run `vagrant up` form within the folder you cloned the repo into

**Note**: Work in progress, modifications will be probably made in the future.
