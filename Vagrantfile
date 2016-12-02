# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ole/jessie64"
  config.vm.box_version = "0.1.6"

  config.vm.hostname = "travis"

  config.vm.define "travis" do |travis|
  end

  config.vm.provider "virtualbox" do |vb|
    vb.name = "travis"
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8082" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8082
  config.vm.network "forwarded_port", guest: 5984, host: 5981, auto_correct: true
  config.vm.network "forwarded_port", guest: 8080, host: 8081, auto_correct: true
  config.vm.network "forwarded_port", guest: 22, host: 2222, host_ip: "0.0.0.0", id: "ssh", auto_correct: true

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "666"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    sudo docker run -d -p 5984:5984 --name bell -v /srv/data/bell:/usr/local/var/lib/couchdb -v /srv/log/bell:/usr/local/var/log/couchdb klaemo/couchdb
    # use crontab to start couchdb on boot
    sudo crontab -l | sudo tee -a mycron
    echo "@reboot sudo docker start bell" | sudo tee -a mycron
    echo "@reboot sudo node /root/server.js" | sudo tee -a mycron
    sudo crontab mycron
    sudo rm mycron
    # install BeLL-Apps
    cd /vagrant
    mkdir -p ole
    cd ole
    git clone https://github.com/open-learning-exchange/BeLL-Apps.git
    cd BeLL-Apps
    git checkout 0.12.65
    chmod +x node_modules/.bin/couchapp

    # create install_linux
    echo "node_modules/.bin/couchapp push \\$1 \\$2" > pushDocToDb.sh
    chmod +x node_modules/.bin/couchapp pushDocToDb.sh
    cp install_windows install_linux
    sed -i "s#pushDocToDb.bat#./pushDocToDb.sh#" install_linux
    #sed -i 's#databases\\\\#databases/#' install_linux
    sed -i 's#databases\\\\\\\\#databases/#' install_linux
    sed -i 's#databases\\\\\\#databases/#' install_linux
    sed -i 's#databases\\\\#databases/#' install_linux
    sed -i 's/NationBell/nation/' init_docs/ConfigurationsDoc-Nation.txt
    sed -i 's/nationbell/NATION/' init_docs/ConfigurationsDoc-Nation.txt
    #echo '{"login": "admin","kind": "Member","roles":["Manager","SuperManager"],"firstName": "Default","lastName": "Admin","password":"password","Gender":"Female","status":"active","email":"admin.nation@olebell.org","visits":0,"bellLanguage":"English","BirthDate":"1995-01-01T00:00:00.000Z","community": "NATION"}' > init_docs/admin.txt
    sed -i 's#Male#Female#' install_linux
    sed -i 's#somalia#nation#' install_linux
    sed -i 's#"visits": 0#"visits":0,"bellLanguage":"English","BirthDate":"2010-10-15T04:00:00.000Z","community":"NATION"#' install_linux

    # check if docker is running
    while ! curl -X GET http://127.0.0.1:5984/_all_dbs ; do
      sleep 1
    done

    # install nation
    node install_linux http://127.0.0.1:5984


    ## fix of log file
    curl -X PUT 'http://127.0.0.1:5984/_config/log/file' -d '"/usr/local/var/log/couchdb/couch.log"'

    ## favicon.ico
    wget https://open-learning-exchange.github.io/favicon.ico
    mv favicon.ico /srv/data/bell/.
    #curl -X PUT 'http://127.0.0.1:5984/_config/httpd_global_handlers/favicon.ico' -d '"{couch_httpd_misc_handlers, handle_favicon_req, \"/usr/local/var/lib/couchdb\"}"'
    curl -X PUT 'http://127.0.0.1:5984/_config/httpd_global_handlers/favicon.ico' -d '"{couch_httpd_misc_handlers, handle_favicon_req, \\"/usr/local/var/lib/couchdb\\"}"'

    # last configurations
    curl -X PUT 'http://127.0.0.1:5984/_config/httpd/allow_jsonp' -d '"true"'
    curl -X PUT 'http://127.0.0.1:5984/_config/admins/nation' -d '"oleoleole"'

    # add redirect on port 8080
    cd /root
    npm install express
    echo '#!/usr/bin/env node' > server.js
    echo '' >> server.js
    echo "var express = require('express')" >> server.js
    echo 'var PortJack = express()' >> server.js
    echo 'PortJack.get(/^(.+)$/, function(req, res) {' >> server.js
    echo 'var options = {' >> server.js
    echo '"127.0.0.1": "http://127.0.0.1:5981/apps/_design/bell/MyApp/index.html",' >> server.js
    echo '"localhost": "http://localhost:5981/apps/_design/bell/MyApp/index.html"' >> server.js
    echo '}' >> server.js
    echo 'if (options.hasOwnProperty(req.hostname)) {' >> server.js
    echo "res.setHeader('Location', options[req.hostname])" >> server.js
    echo '}' >> server.js
    echo 'else {' >> server.js
    echo "res.setHeader('Location', 'http://ole.org')" >> server.js
    echo '}' >> server.js
    echo 'res.statusCode = 302' >> server.js
    echo 'res.end()' >> server.js
    echo '})' >> server.js
    echo 'PortJack.listen(8080)' >> server.js
    chmod +x server.js
    node /root/server.js &
  SHELL
end
