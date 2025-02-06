Vagrant.configure("2") do |config|

    config.vm.box = "debian/bullseye64"
    config.vm.network "forwarded_port", guest:5000, host:5000

    config.vm.provision "shell", inline: <<-SCRIPT
  
    # Install services
    apt-get update && sudo apt-get install -y python3-pip nginx

    # Start Nginx Server
    systemctl start nginx

    # Work directory and permission
    mkdir -p /var/www/myapp
    chown -R vagrant:www-data /var/www/myapp
    chmod -R 775 /var/www/myapp
    cp /vagrant/files/.env /var/www/myapp/.env

    

    SCRIPT

    config.vm.provision "shell", privileged: false, inline: <<-SHELL
    # Install services
    pip3 install pipenv
    pip3 install python-dotenv

    # Start environment
    cd /var/www/myapp
    pipenv shell

    # Install Flask and Gunicorn services in virtual environment
    pipenv install flask gunicorn

    # App files
    sudo cp /vagrant/files/application.py /var/www/myapp/application.py 
    sudo cp /vagrant/files/wsgi.py /var/www/myapp/wsgi.py 

    SHELL
  
  end
  