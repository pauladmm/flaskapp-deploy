Vagrant.configure("2") do |config|

    config.vm.box = "debian/bullseye64"
    
    config.vm.provision "shell", inline: <<-SCRIPT
  
    # Install services
    sudo apt-get update && sudo apt-get install -y python3-pip

    # Work directory and permission
    mkdir -p /var/www/myapp
    chown -R vagrant:www-data /var/www/myapp
    chmod -R 775 /var/www/myapp
    cp /vagrant/files/.env /var/www/myapp/.env

    # 

    SCRIPT

    config.vm.provision "shell", privileged: false, inline: <<-SHELL
    # Install services
    pip3 install pipenv
    pip3 install python-dotenv

    SHELL
  
  end
  