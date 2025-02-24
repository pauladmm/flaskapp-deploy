Vagrant.configure("2") do |config|

    config.vm.box = "debian/bullseye64"
    config.vm.network "forwarded_port", guest:5000, host:5000
    config.vm.network "private_network", ip: "192.168.56.10"

    config.vm.provision "shell", inline: <<-SCRIPT
  
    # Install services
    apt-get update && sudo apt-get install -y python3-pip nginx git

    # Start Nginx Server
    systemctl start nginx

    # MYAPP Work directory and permission 
    mkdir -p /var/www/myapp
    chown -R vagrant:www-data /var/www/myapp
    chmod -R 775 /var/www/myapp

    # App cloned & permission & files
    cd /var/www/
    git clone https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart
    chown -R vagrant:www-data /var/www/msdocs-python-flask-webapp-quickstart
    chmod -R 775 /var/www/msdocs-python-flask-webapp-quickstart
    cp /vagrant/files/msdocs-webapp/wsgi.py /var/www/msdocs-python-flask-webapp-quickstart/wsgi.py
    cp /vagrant/files/msdocs-webapp/msdocs_app.service /etc/systemd/system/msdocs_app.service

    # MYAPP copy files of virtual environment and gunicorn
    cp /vagrant/files/.env /var/www/myapp/.env
    cp /vagrant/files/flask_app.service /etc/systemd/system/flask_app.service
    
    # Nginx Server Configuration
    cp /vagrant/files/myapp.conf /etc/nginx/sites-available/myapp.conf
    ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/

    cp /vagrant/files/msdocs-webapp/msdocs.conf /etc/nginx/sites-available/msdocs.conf
    ln -s /etc/nginx/sites-available/msdocs.conf /etc/nginx/sites-enabled/


    SCRIPT

    config.vm.provision "shell", privileged: false, inline: <<-SHELL
    # Install services
    pip3 install --user pipenv
    pip3 install --user python-dotenv

    # Agregar ~/.local/bin a PATH para que reconozca pipenv
    echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
    export PATH=$HOME/.local/bin:$PATH

    # Start virtual environment
    cd /var/www/myapp
    pipenv --python 3
    

    # Install Flask and Gunicorn services in virtual environment
    pipenv install flask gunicorn

    # MYAPP files
    sudo cp /vagrant/files/application.py /var/www/myapp/application.py 
    sudo cp /vagrant/files/wsgi.py /var/www/myapp/wsgi.py

    # Msdocs-webapp virtual environment activation and requirements installation
    cd /var/www/msdocs-python-flask-webapp-quickstart/
    pipenv --python 3
    pipenv install -r requirements.txt

    # Reload system
    sudo systemctl daemon-reload
    sudo systemctl enable flask_app
    sudo systemctl enable msdocs_app
    sudo systemctl start flask_app
    sudo systemctl start msdocs_app
    sudo systemctl restart nginx
    SHELL
  
  end
  