# PROJECT SUM UP

Development of a web application using Flask as the framework and Gunicorn as the application server. The app runs on a virtual machine provisioned with Vagrant

# TECHNOLOGIES USED

- Vagrant (to provision the virtual machine)
- Nginx (server)
- Python3
- Pipenv package (virtual environment management)
- Python-dotenv package (environment variables)
- Flask (framework)
- Gunicorn (application server)

# CREATION AND DEPLOYMENT OF AN APP WITH FLASK

## Set Up Environment

The first thing to set up the environment is to install the services. It is important not to install pip3 packages with administrator priviledges due to security issues.
In the Vagrantfile two blocks of provision are used to specify the commands ran with priviledges or not.
We use this command to assure pipenv is correctly installed in the user directory:

```
PATH=$PATH:/home/$vagrant/.local/bin pipenv --version
```

The work directory is created: myapp
It is mandatory to give permissions to this directory in order to allow Nginx to access to this app (in following steps).

## App Creation

Before the app creation, the environment variables must be stored in an .env directory file, inside the app directory (var/www/myapp).
The content of this file should be:

```
FLASK_APP=wsgi.py
FLASK_ENV=production

```

wsgi.py will tell the app how to comunicate with the server
production is the app state

After this the virtual environment can be started with

```
pipenv shell
```

And it is showed as:
![pipenv shell](/img/pipenv-shell.PNG)

## App Deployment with Flask and Gunicorn

Inside the virtual environment, Flask and Gunicorn are installed.

```
pipenv install flask gunicorn
```

Two files inside the virtual environment are created: application.py and wsgi.py. This two files contain information about the App (Proof of Concept App)

- application.py will contain the app itself.
- wsgi.py will contain the way to run it.

Yet inside the virtual environment, the app is runned the following way, to host it in any IP available:

```
flask run --host '0.0.0.0'
```

It can be accessed in http://IP-maq-virtual:5000, in this case the url was http://127.0.0.1:5000

![App Deployed](/img/app-deployed.PNG)

To check the deployment with Gunicorn this command was used (inside the virtual environment):

```
gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app
```

And in the same url the app can be seen.
![App Deployed with Gunicorn](/img/gunicorn-app-deployed.PNG)
![App Deployed](/img/app-deployed.PNG)

This app is runned trough a path that can be checked with:

```
which gunicorn
```

In this case:

```
/home/vagrant/.local/share/virtualenvs/myapp-UTYI2jO8/bin/gunicorn
```

# DEPLOYMENT WITH NGINX AND GUNICORN

Gunicorn must be runned as another service inside the virtual machine, so the next file must be created in etc/systemd/system/flask_app.service:

```
[Unit]
Description=flask app service - App con flask y Gunicorn
After=network.target
[Service]
User=vagrant
Group=www-data
Environment="PATH=/home/vagrant/.local/share/virtualenvs/myapp-UTYI2jO8/bin/gunicorn"
WorkingDirectory=/var/www/myapp
ExecStart=/home/vagrant/.local/share/virtualenvs/myapp-UTYI2jO8/bin/gunicorn --workers 3 --bind unix:/var/www/myapp/app.sock wsgi:app

[Install]
WantedBy=multi-user.target
```

The path and the directore of the App created previously must be specified.

The system must be reloaded to add this service and the service must be started:

```
systemctl daemon-reload

systemctl enable flask_app
systemctl start flask_app
```

The name of the service is the same as the file created previously (flask_app).

## Nginx Server Configuration

To configure Nginx, a file .conf should be created with the name of the server and the proxy pass as it showed below:

```
server {
  listen 80;
  server_name myapp.es www.myapp.es;

  access_log /var/log/nginx/app.access.log;
  error_log /var/log/nginx/app.error.log;

  location / {
    include proxy_params;
    proxy_pass http://unix:/var/www/myapp/app.sock;
  }
}
```

The proxy pass block is where Nginx understand it has to do inverse proxy to the socket created by Gunicorn, in order to access the Flask App created.

The next steps are basic Nginx configuration:

1. Copy this file in /etc/nginx/sites-available
2. Enable symbolic link in /etc/nginx/sites-enabled/

![Symbolic Link Comprobation](/img/check-symbolic-link-nginx.PNG)

## App Access

Due to Gunicorn is serving the App, it must be accessed by the server_name. To do that, the hosts file (Windows->System32->drivers->etc>hosts) should be modified to add the VM IP and associate it with the server_name.
In this case:

```
192.168.56.10  myapp.es www.myapp.es
```

Once the MV IP is linked to this server name in the file hosts in the host, the app can be accesses through the navigator in myapp.es or www.myapp.es

![App Deployed with Nginx](/img/app-deployed-nginx.PNG)

# EXTRA: APP CLONED (FROM GITHUB REPO) DEPLOYMENT

The objective is deploy another app, but this time the app is cloned directly from a github repository.
The App is https://github.com/Azure-Samples/msdocs-python-flask-webapp-quickstart.

The steps are very similar to the previous. The **App would be cloned** in /var/www/ and in this directory, **permission** would be given to this new directory and the **virtual environment should be activate**. Then the **dependencies would be installed** through this command, inside the virtual environment:

```
pipenv install -r requirements.txt
```

A wsgi.py file must be created (this time _from app_ because the app name file is app.py). This file is saved in the same directory as the app.py (/var/www/msdocs-python-flask-webapp-quickstart/)

```
from app import app

if __name__ == "__name__":
    app.run(debug=False)

```

## Gunicorn Set Up

This command will be runned inside the virtual environment to check that gunicorn is activated:

```
gunicorn --workers 4 --bind 0.0.0.0:5000 wsgi:app
```

![Msdocs App Deployed with Gunicorn](/img/msdocs-app-gunicorn-deployed.PNG)

To know the path gunicorn is executing, the comand which gunicorn is used again. this time the path is:

```
/home/vagrant/.local/share/virtualenvs/msdocs-python-flask-webapp-quickstart-YrxIX3KC/bin/gunicorn
```

## Nginx Deployment

A new .service file is needed, using the previous path. The file is created in /etc/systemd/system/:

```
[Unit]
Description=Msdocs Python Flask Webapp Quickstart
After=network.target
[Service]
User=vagrant
Group=www-data
Environment="PATH=/home/vagrant/.local/share/virtualenvs/msdocs-python-flask-webapp-quickstart-YrxIX3KC/bin/gunicorn"
WorkingDirectory=/var/www/msdocs-python-flask-webapp-quickstart
ExecStart=/home/vagrant/.local/share/virtualenvs/msdocs-python-flask-webapp-quickstart-YrxIX3KC/bin/gunicorn --workers 3 --bind unix:/var/www/msdocs-python-flask-webapp-quickstart/app.sock wsgi:app

[Install]
WantedBy=multi-user.target
```

Also, a configuration file with a server_name and the proxy_pass path. The file is created in /etc/nginx/sites-available and the symbolic link is created.

```
server {
  listen 80;
  server_name msdocs.es www.msdocs.es;

  access_log /var/log/nginx/app.access.log;
  error_log /var/log/nginx/app.error.log;

  location / {
    include proxy_params;
    proxy_pass http://unix:/var/www/msdocs-python-flask-webapp-quickstart/app.sock;
  }
}
```

msdocs_app.service should be enabled and started, then the services should be restarted (daemon-reload and nginx).
Hosts file should be updated with the new server_name.
The app will be available in any of the names given.

![Msdocs App Deployed with Nginx](/img/msdocs-app-nginx.PNG)
