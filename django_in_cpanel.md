# Deploying Django in cPanel

A guide for deploying a Django project into a cPanel shared environment. These steps outline my general approach to
deploying into a subdomain. Additional information is given at the end for publishing into the main domain.

## Prerequisites

Ensure that your chosen hosting package includes access to the following cPanel tools

- Terminal
- Setup Python App
- File Manager
- Domains
- MySQL&#174; Databases

_Note: cPanel Hosting providers may provide Terminal and Setup Python App in their basic hosting package, which is often
aimed at WordPress users only. However, choosing the next package up typically brings benefits in terms of the number of
projects / databases / storage capacity etc._

## Set up the subdomain

If not already setup, use the cPanel Domains tool to create the subdomain, making a note of the path to the `document
root`. This is where the django project will be installed.

### Check

- Using a web browser, ensure that the subdomain url is accessible. By default, it should return the contents of
  the `document root`

## Set up the Python app

Use the cPanel Setup Python App to create the application.

_Note: cPanel are slow when it comes to supporting the latest version of `mysqlclient`. At the time of writing it
appears that `mysqlclient==2.1.1`  is only supported on `python 3.8`_

Use the following field values:

- Python version: _3.8.n_ (choose the latest 3.8 version available from the drop-down list)
- Application root: the document root for your subdomain
- Application URL: choose the url for your subdomain, from the drop-down
- Application startup file: `application`
- Application Entry point: `passenger_wsgi.py`

Click on `CREATE`

Click on `START APP`

Again, using a web browser, ensure that the subdomain url is accessible. By default, it should now show...

```
It works!
Python 3.8.n
```

This means a virtual environment for the Python application has been successfully created.

### Opening a project virtual environment

Edit the Python application (use the pencil icon). Towards the top of the edit screen the command to access the
virtual environment via the cPanel Terminal app will be displayed
e.g.

To enter to virtual environment, open a cPanel Terminal tool and run the command copied from the python app.
e.g. command: `source /home/<username>/virtualenv/<subdomain>/3.8/bin/activate && cd /home/<username>/<subdomain>`

## Uploading project files

There are a number of ways of uploading project files e.g. using a `FTP` client, `clone` from Github and then copy to
the project directory, or even use cPanel File Manager

The key thing is that the project directory/folder containing `manage.py` should sit directly below the `document root`
of the subdomain.

For example this is what the project structure would look like for
the [django polls project](https://docs.djangoproject.com/en/5.0/intro/tutorial01/) ...

```
/home/<username>/<directory root>/
  manage.py
  README.md
  requirements.txt
  mysite/
    settings.py
    urls.py
    wsgi.py
    asgi.py
    __init__.py
  polls/
   __init__.py
   admin.py
   apps.py
   migrations/
    __init__.py
   models.py
   tests.py
   views.py 
```

## Production settings

For application security the `settings.py` file used in production must not contain development server settings.

There are a number of ways of doing this; a separate `settings.py`, overwriting the `seetings.py` by copying a
production `prod_settings.py` onto `settings.py`, using a combination of settings files
e.g. `base.py`, `dev.py`, `prod.py`

_Note: the combination of settings files is favoured by coderedcms_

Whichever method is used, the resulting `settings.py` needs to include the following settings, where environmental
variables are imported from a local `.env` file...

```
# Initialise environment variables
env = environs.Env()
environs.Env.read_env()

PROJECT_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
BASE_DIR = os.path.dirname(PROJECT_DIR)

# Used for cPanel deployments into main domain
PUBLIC_HTML = os.path.expanduser("~/public_html")

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = env("SECRET_KEY")

# Add your site's domain name(s) here.
ALLOWED_HOSTS = env.list("ALLOWED_HOSTS")

STATIC_ROOT = os.path.join(BASE_DIR, "static")
STATIC_URL = "/static/"

MEDIA_ROOT = os.path.join(BASE_DIR, "media")
MEDIA_URL = "/media/"

DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.mysql",
            "NAME": env("DATABASE_NAME"),
            "USER": env("DATABASE_USER"),
            "PASSWORD": env("DATABASE_PASS"),
            "HOST": "localhost",
            "PORT": "3306",
            "OPTIONS": {
                "init_command": "SET sql_mode='STRICT_TRANS_TABLES'",
            },
        }
    }

MIDDLEWARE = [
    .
    .
    .
    "django.middleware.security.SecurityMiddleware",
    # Uncomment for serving static files in prod using whitenoise - must come after SecurityMiddleware
    "whitenoise.middleware.WhiteNoiseMiddleware",
    .
    .
    .
]

```

### Example .env

The `.env` file should be placed in the same directory as `settings.py`

```
ALLOWED_HOSTS=<subdomain>.<your domain>,www.<subdomain>.<your domain>
SECRET_KEY=<django secret key>
DATABASE_NAME=<db name>
DATABASE_USER=<db username>
DATABASE_PASS=<db password>
```

## Installing Python packages

In a similar way to `settings.py`, there are a number of ways of installing the production requirements. In this guide,
we will use a separate `prod-requirements.txt`

Assuming that coderedcms is being used, the `prod-requirements.txt` file should look similar to the example below.

_Note: `whitenoise` and `mysqlclient` are required in production._

```
coderedcms==2.1.*
wagtail==4.2.*
requests==2.31.0
environs==9.5.0
mysqlclient==2.1.1
whitenoise~=6.6.0
.
.
.
```

To install the pre-requisites, open a project virtual environment (using the method described above)

Ensure that `pip` is up to-date

`pip install --upgrade pip`

then install the product requirements - in our case from `prod-requirements.txt`

`pip install -r prod-requirements.txt`

## Setting up the database tables

To setup the database tables required for the project and related 3rd party apps run the following command.

`python manage.py migrate`

Once the database has been created, the first user needs to be added - this is always a 'superuser'

Use the command `python manage.py createsuperuser` and follow the prompts - make sure to record the credentials used!

## Gathering up the static files

In a production environment, all the django static files are collected from their respective directories and added to
a directory directly under the `document root`

To do this run the following command ..

`python manage.py collectstatic`

Type ‘yes’ if prompted

After a while you will see something like:

`132 static files copied to '/home/<username>/<document root>/static'`

## Connecting the Python app to the Django project

By default, when a Python app is created it creates a `passenger_wsgi.py` file in `document_root`. This file needs to be
updated to specify that the application is a django application and also specify the location of the `settings.py file
to be used.

Example `passenger_wsgi.py`

```
"""
Passenger file for running Django in cPanel
"""
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "<settings directory>.settings")
application = get_wsgi_application()
```

## One final check

Before starting/re-starting the Python App in the Setup Python App tool, I start the app locally in
the `project virtual environment` using the following command...

`python manage.py runserver`

I find it very useful for quickly exposing any fundamental issues e.g steps misssed etc.

## And finally...

Once satisfied, use the cPanel Setup Python App to restart the application. Then, use your browser to confirm that you
can access the website.

## Troubleshooting

A useful source of information, should you encounter problems when accessing the website e.g. `500 errors`, `400 errors` etc. is to look in the `stderr.log` in the `document root` directory.

After that - it's Stackoverflow, Google etc.


## Additional instructions for deploying to the main domain

Within cPanel the `document root` of the main domain is automatically set to `/pubic_html/` However, Python apps cannot be installed directly into `/public_html` so the `settings.py` needs to be tweaked to indicate this.

In the original `setting.py` example above the `BASE_DIR` setting was used to specify the location of `static` and `media`

```
.
.
.
PROJECT_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
BASE_DIR = os.path.dirname(PROJECT_DIR)

# Used for cPanel deployments into main domain
PUBLIC_HTML = os.path.expanduser("~/public_html")
.
.
.
STATIC_ROOT = os.path.join(BASE_DIR, "static")
STATIC_URL = "/static/"

MEDIA_ROOT = os.path.join(BASE_DIR, "media")
MEDIA_URL = "/media/"
.
.
.
```

When using the main domain, the settings file should be changed as follows...

```
PROJECT_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
BASE_DIR = os.path.dirname(PROJECT_DIR)

# Used for cPanel deployments into main domain
PUBLIC_HTML = os.path.expanduser("~/public_html")

STATIC_ROOT = os.path.join(BASE_DIR, "static")
STATIC_URL = "/static/"

MEDIA_ROOT = os.path.join(PUBLIC_HTML, "media")
MEDIA_URL = "/media/"
```

----------------END-------------------













 

