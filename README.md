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
