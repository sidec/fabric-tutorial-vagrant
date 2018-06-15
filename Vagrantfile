# -*- mode: ruby -*-
# vi: set ft=ruby :
# Vagrant provisioning script for creating a standalone hyperledger fabric and composer environment
# Copyright (C) 2017 Suen Chun Hui

# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.

# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.

# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

# VAGRANTFILE_API_VERSION = "2"
ARCH = "x86_64"
FABRIC_DOCKER_VER = "1.1.0"
COMPOSER_VER = "0.19.9"
FABRIC_SAMPLE_VER = "v1.0.2"

$script = <<SCRIPT
set -x
apt-get update
apt-get -y install docker.io docker-compose git openssl golang-go libltdl-dev
addgroup ubuntu docker
echo "#!/bin/bash" > /etc/rc.local
echo "service docker start" >> /etc/rc.local
service docker start

#fabric
docker pull hyperledger/fabric-peer:#{ARCH}-#{FABRIC_DOCKER_VER}
docker pull hyperledger/fabric-ca:#{ARCH}-#{FABRIC_DOCKER_VER}
docker pull hyperledger/fabric-ccenv:#{ARCH}-#{FABRIC_DOCKER_VER}
docker pull hyperledger/fabric-orderer:#{ARCH}-#{FABRIC_DOCKER_VER}
docker pull hyperledger/fabric-couchdb:#{ARCH}-#{FABRIC_DOCKER_VER}
docker pull hyperledger/fabric-tools:#{ARCH}-#{FABRIC_DOCKER_VER}
docker tag hyperledger/fabric-peer:#{ARCH}-#{FABRIC_DOCKER_VER} hyperledger/fabric-peer:latest
docker tag hyperledger/fabric-ca:#{ARCH}-#{FABRIC_DOCKER_VER} hyperledger/fabric-ca:latest
docker tag hyperledger/fabric-ccenv:#{ARCH}-#{FABRIC_DOCKER_VER} hyperledger/fabric-ccenv:latest
docker tag hyperledger/fabric-orderer:#{ARCH}-#{FABRIC_DOCKER_VER} hyperledger/fabric-orderer:latest
docker tag hyperledger/fabric-couchdb:#{ARCH}-#{FABRIC_DOCKER_VER} hyperledger/fabric-couchdb:latest
docker tag hyperledger/fabric-tools:#{ARCH}-#{FABRIC_DOCKER_VER} hyperledger/fabric-tools:latest

#composer
docker pull hyperledger/composer-playground:#{COMPOSER_VER}
docker pull hyperledger/composer-rest-server:#{COMPOSER_VER}
docker pull hyperledger/composer-cli:#{COMPOSER_VER}
docker tag hyperledger/composer-playground:#{COMPOSER_VER} hyperledger/composer-playground:latest
docker tag hyperledger/composer-rest-server:#{COMPOSER_VER} hyperledger/composer-rest-server:latest
docker tag hyperledger/composer-cli:#{COMPOSER_VER} hyperledger/composer-cli:latest

#clone fabric repo & build cryptogen configtxgen
cd / ; git clone https://github.com/hyperledger/fabric -b v#{FABRIC_DOCKER_VER}
cd / ; mkdir -p go/src/github.com/hyperledger; cd go/src/github.com/hyperledger ; ln -s /fabric . ; cd fabric ; GOPATH=/go make cryptogen configtxgen
cd / ; cp /fabric/build/bin/cryptogen /usr/bin/ ; cp /fabric/build/bin/configtxgen /usr/bin/

#clone sample repo
su ubuntu -c "bash -c 'cd; git clone https://github.com/hyperledger/fabric-samples -b #{FABRIC_SAMPLE_VER}'"

#npm, yeoman tools & composer rest server
apt-get install -y npm
npm install -g n
n 6.9
rm -rf /usr/bin/node /usr/bin/nodejs
cd /usr/local/bin/ ; ln -s node nodejs
npm install -g yo generator-fountain-webapp http-server@0.10.0 typings bower @angular/cli generator-hyperledger-composer composer-cli@0.16.3 composer-rest-server@0.16.3

#composer playground
su ubuntu -c "cd ; git clone https://github.com/suenchunhui/fabric-tutorial-vagrant ; ln -s fabric-tutorial-vagrant/composer-playground ."
su ubuntu -c "cd ; cd composer-playground ; chmod a+x playground.sh ; chmod a+x scripts/*.sh ; cd fabric-composer-tools ; docker build -t fabric-composer-tools ."

#cloud9 IDE
apt-get install -y curl build-essential nodejs
curl -sL https://deb.nodesource.com/setup_6.x | bash -
cd /usr/bin/ && ln -s nodejs node
cd / ; git clone https://github.com/c9/core.git cloud9
cd /cloud9 && scripts/install-sdk.sh
chmod a+rw -R /cloud9/build
echo 'cd /cloud9 ; su ubuntu -c "screen -d -m nodejs server.js -l 0.0.0.0 -w /home/ubuntu --auth root:secret"' >> /etc/rc.local
cd /cloud9 ; su ubuntu -c "screen -d -m nodejs server.js -l 0.0.0.0 -w /home/ubuntu --auth root:secret"

echo "exit 0" >> /etc/rc.local
SCRIPT


Vagrant.configure('2') do |config|
  config.vm.box = "generic/ubuntu1604"
  config.vm.provider "hyperv"
  config.vm.synced_folder ".", "/vagrant", type: "smb"
  config.vm.provision "shell", inline: $script
  config.vm.network "forwarded_port", guest: 8080, host: 8080  #composer
  config.vm.network "forwarded_port", guest: 8181, host: 8181  #cloud9-ide
  config.vm.network "forwarded_port", guest: 9090, host: 9090  #custom-ui
  config.vm.network "forwarded_port", guest: 3000, host: 3000  #marbles-ui
  config.vm.network "forwarded_port", guest: 3001, host: 3001  #marbles-ui

  config.vm.provider "hyperv" do |h|
    h.enable_virtualization_extensions = true
    h.differencing_disk = true
  end

end
