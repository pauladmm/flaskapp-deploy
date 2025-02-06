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

## Create the App

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
