VAGRANTFILE_API_VERSION = "2"

$install_docker = <<eos
for i in 1 2 3; do apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D && break || sleep 15; done
echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list
apt-get update -y && apt-get purge -y lxc-docker*
apt-get -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" install -y docker-engine
curl --silent -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose
gpasswd -a vagrant docker && service docker restart
eos

$setup_vm = <<eos
grep -r "^cd /var/www/html" /home/vagrant/.bashrc || echo "cd /var/www/html" >> /home/vagrant/.bashrc
eos

$run_docker = <<eos
service docker restart
cd /var/www/html
docker-compose up -d
eos

$provision_app = <<eos
cd /var/www/html && docker-compose run --rm server create_db && docker-compose run --rm postgres psql -h postgres -U postgres -c "create database tests"
eos

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "dev-redash-site"
  config.vm.network :private_network, ip: "192.168.3.62"

  config.vm.synced_folder ".", "/var/www/html", :mount_options => ['dmode=777,fmode=777']

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "3072"]
    vb.customize ["modifyvm", :id, "--rtcuseutc", "on"]
    vb.customize ["modifyvm", :id, "--cpus", 4]
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/var/www/html", "1"]
  end

  config.vm.provision "fix-no-tty", type: "shell" do |s|
    s.privileged = false
    s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
  end

  config.vm.provision :shell, inline: $install_docker
  config.vm.provision :shell, inline: $setup_vm, run: "always"
  config.vm.provision :shell, inline: $run_docker, run: "always"
  config.vm.provision :shell, inline: $provision_app
  config.vm.provision :shell, inline: "echo -e \"\e[0m\e[7m\e[32m------- Everything is READY! -------\e[0m\"", run: "always"
end
