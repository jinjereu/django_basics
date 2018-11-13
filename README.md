# django_basics

This is a project following the Django Basics tutorial from <https://docs.djangoproject.com/en/2.1/intro/tutorial01/>

Requirements: 
* Python
* Supports Python 2.x to 3.x

## Installation of Django in Virtual Environment 

First follow installation instructions for virtualenvwrapper in <https://virtualenvwrapper.readthedocs.io/en/latest/> and install django on a specific working environment.

## Part 1 Summary: App Start-up and Basic Views

* Determine Django version

`python -m django --version`

### Creating a project

* Auto generate code to establish a Django project which consists of a collection of settings for an instance of Django, database configuration, Django- and application-specific settings 

1. cd to the directory where you like to story your code
2. Run `django-admin startproject mysite`

This will create the following:

```
mysite/
	manage.py
		mysite/
		__init__.py
		settings.py
		urls.py
		wsgi.py
```

These files are: 
* `mysite/` - root directory and container for project 
* `manage.py` - cmd line utility to interact with Django project
* inner `mysite/` - Python package for the project. When importing to other places referenced as `import mysite.urls` for example.
* `mysite/__init__.py` - Empty file to denote this directory is a Python package
* `mysite/settings.py` - Configurable Django project settings
* `mysite/urls.py` - URL declarations for the project 
* `mysite/wsgi.py` - entry point 

### The development server 

The development server automatically reloads Python code for each request as needed. Usually you don't need to restart the server, except for some actions like adding files.  

* Run the development server (By default runs at port 8000)

`python manage.py runserver`


* Running the development server on a specific port for ex `8080`:

`python manage.py runserver 8080`


* Running the development server on a specific `ip` and `port`: 

`python manage.py runserver <ip>:<port>`

### Creating the `Polls` app

*Apps vs Projects*
* Apps - web application that does something i.e. log system, database of records, etc. Can be in multiple projects. Apps can live anywhere on your Python path. 
* Projects - collection of configuration and apps for a particular website.

* To create an app named `polls`:

`python manage.py startapp polls`

This command creates the ff structure: 

```
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

### Recipe: Writing Views

Method A:

1. For example you have a view `/results`. Open the `polls/views.py` file and add your definitions. 

```
def results():
	return HttpResponse("Hello, world.")
```

2. To call the view, map it to a URL. Open `polls/urls.py` and add a URL configuration. See more info on defining a [path()]

```
urlpatterns = [
	path('', views.result, name='result')
]
```
[path()]: https://docs.djangoproject.com/en/2.1/ref/urls/#django.urls.path

3. Point the root URLconf at the `polls.urls` module in `mysite/urls.py` by using the `include()` function

```
urlpatterns = [
	path('polls/', include('polls.urls')),
	path('admin/', admin.site.urls),
]
```

## Part 2 Summary: Database, Models and Admin introduction

### Database setup

By default the configuration uses SQLite. The configuration can be found in `<project: mysite>/settings.py` in the `DATABASES` settings configuration.

If you not using SQLite, additional settings such as ` USER` , `PASSWORD`, `HOST` must be added. See reference documentation for [DATABASES]

[DATABASES]: https://docs.djangoproject.com/en/2.1/ref/settings/#std:setting-DATABASES

1. Edit `TIME_ZONE` to your time zone
2. Run `python manage.py migrate` - looks at `INSTALLED_APPS` setting to and creates the necessary db settings
3. Tell the project that the `polls` app is installed. Update `mysite/settings.py` and add the polls config app to the `INSTALLED_APPS` setting. 
```
INSTALLED_APPS = [
'polls.apps.PollsConfig',
...
]
```

### Recipe: Models

A model is a single source of truth about the fields and behaviors of the data you are storing. Django follows DRY Principle - defining data model in one place and derive all things from it.  

Django will derive from the model to create the DB schema and create Python DB access API.

1. Change your models. 
* Edit the `<app: polls>/models.py` adding a `Class` definition for each entity you want represented in the application. 
* Add the fields represented by an instance of a `Field` class. Note that some `Field` classes have required arguments.  Define relationships with `ForeignKey()`
* Add relevant methods to your model

```
from django.db import models

class Question(models.Model):
	question_text = models.CharField(max_length=200)
	pub_date = models.DateTimeField('date published')
	
	def was_published_recently(self):
	return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
	
class Choice(models.Model):
	question = models.ForeignKey(Question, on_delete=models.CASCADE)
	choice_text = models.CharField(max_length=200)
	votes = models.IntegerField(default=0)
```
2. Create migrations for changes:  
`python manage.py makemigrations <appname: polls>`

3. Apply changes to the database:
`python manage.py migrate`

4. If you want to make the app's models modifiable in the admin page we need to register the model in `<app: polls>/admin.py`

```
from .models import Question

admin.site.register(Question)
```

### Creating an admin user

The Django automates the creation of admin interfaces for models. The Django admin site is activated by default 

1. Create a user to login to the admin site, enter desired username, email address and password
`python manage.py createsuperuser`
2. Start the development server 
`python manage.py runserver`
3. Access the admin site at `<http:>//<domain>/admin/`












