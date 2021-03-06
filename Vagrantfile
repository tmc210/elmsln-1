# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # centos 7.2 - 64 bit
  #config.vm.box = "bradallenfisher/centos7"
  # primed ubuntu 16.04 - 64 bit
  config.vm.box = "elmsln/ubuntu16"
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end
  # private network port maping, host files point to elmsln domains
  config.vm.network "private_network", ip: "10.0.18.55"
  # forward the vm ports for database and apache to local ones
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.network "forwarded_port", guest: 3306, host: 3306
  config.vm.hostname = "elmsln.local"
  config.hostsupdater.aliases = ["courses.elmsln.local", "media.elmsln.local", "online.elmsln.local", "analytics.elmsln.local", "studio.elmsln.local", "interact.elmsln.local", "blog.elmsln.local", "comply.elmsln.local", "discuss.elmsln.local", "inbox.elmsln.local", "people.elmsln.local", "innovate.elmsln.local", "grades.elmsln.local", "hub.elmsln.local", "lq.elmsln.local", "cdn1.elmsln.local", "cdn2.elmsln.local", "cdn3.elmsln.local", "data-courses.elmsln.local", "data-media.elmsln.local", "data-online.elmsln.local", "data-studio.elmsln.local", "data-interact.elmsln.local", "data-blog.elmsln.local", "data-comply.elmsln.local", "data-discuss.elmsln.local", "data-inbox.elmsln.local", "data-people.elmsln.local", "data-innovate.elmsln.local", "data-grades.elmsln.local", "data-hub.elmsln.local", "data-lq.elmsln.local"]
  config.vm.boot_timeout = 600
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
  # automatically carve out 1/4 of RAM for this VM
  config.vm.provider "virtualbox" do |v|
    host = RbConfig::CONFIG['host_os']

    # Give VM 1/4 system memory or minimum 2 gigs
    if host =~ /darwin/
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
    elsif host =~ /linux/
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
    elsif host =~ /mingw32/
      mem = `wmic os get TotalVisibleMemorySize | grep '^[0-9]'`.to_i / 1024 / 4
      if mem < 2048
        mem = 2048
      end
    else # sorry weird Windows folks, I can't help you
      mem = 2048
    end
    # you can modify these manually if you want specific specs
    v.customize ["modifyvm", :id, "--memory", mem]
    v.customize ["modifyvm", :id, "--cpus", 1]
    v.name = "elmsln"
  end
  # https://github.com/hashicorp/vagrant/issues/7508
  # fix for ioctl error
  config.vm.provision "fix-no-tty", type: "shell" do |s|
    s.privileged = true
    s.inline = "sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n \\|\\| true/' /root/.profile"
  end
  # fix apt lock error
  config.vm.provision "disable-apt-periodic-updates", type: "shell" do |s|
    s.privileged = true
    s.inline = "echo 'APT::Periodic::Enable \"0\";' > /etc/apt/apt.conf.d/02periodic"
  end
  # run script as root
  config.vm.provision "shell",
    args: "https://github.com/elmsln/elmsln.git 0.11.x https://github.com/elmsln/elmsln-config-vagrant.git",
    path: "scripts/vagrant/handsfree-vagrant.sh"

  # run as the vagrant user
  config.vm.provision "shell",
    path: "scripts/vagrant/cleanup.sh",
    privileged: FALSE

  # all done! tell them how to login
  config.vm.provision "shell",
    inline: "echo 'finished! go to http://online.elmsln.local and login with username admin and password admin. To edit files on the box point an sftp client to /var/www/elmsln on 0.0.0.0 u/p vagrant/vagrant port 2222.'"
end
