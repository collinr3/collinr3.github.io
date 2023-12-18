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

Now edit the Python application (use the pencil icon). Towards the top of the edit screen the command to access the
virtual environment via the cPanel Terminal app will be displayed
e.g.

To enter to virtual environment, run the
command: `source /home/<username>/virtualenv/<subdomain>/3.8/bin/activate && cd /home/<username>/<subdomain>`

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





 

