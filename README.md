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

## Part 3 Summary: 

### Writing views with arguments and raising exception


1. You can also write views with an argument which you provide through the URL. For example, in `polls/views.py` 

```
from django.shortcuts import get_object_or_404, render

def detail(request, question_id):
	# It is very common to raise HTTP404 exception so this shortcut can be used
	# There is also a get_list_or_404() method for lists
	question = get_object_or_404(Question, pk=question_id)
	return render(request, 'polls/detail.html', {'question': question})


def results(request, question_id):
	response = "You're looking at the results of question %s."
	return HttpResponse(response % question_id)
```



2. Wire the views to `polls.urls`. Also add the namespace via `appname=polls` to differentiate it from other apps in the project. 

```
appname = `polls`
urlpatterns = [
	...
	# ex: /polls/5/
	path('<int:question_id>/', views.detail, name='detail'),
	# ex: /polls/5/results/
	path('<int:question_id>/results/', views.results, name='results'),
]
```

Using angle brackets as in `<int:question_id>` "captures" part of the URL and sends it as a keyword argument to the view function. 
- The `question_id` part of the string defines the name that will be used to identify the matched pattern.
- The `<int:` paer is a converter that determines what patterns should match this part of the URL path

### Writing views and output to template

`TEMPLATES` setting describes how it will render templates and by convention it looks for a `templates` subdirectory in each of the `INSTALLED_APPS`.  

Python code is declared in template as `{% %}` 
Meanwhile to access variable attributes, use the dot-lookup syntax for ex. `{{ question.question_text }}`

1. Create directory `templates` in `polls` directory. 

This is where Django will look for the templates. 

2.  Within the `templates` directory, create another directory called `polls` and within that create a file called `index.html`

The full url is `polls/templates/polls/index.html`
This is how `app_directories` template loader will locate the templates as described above. 
But you can refer to it in Django simply as `polls.index.html` 

3. Update `index.html` template 

```
{% if latest_question_list %}
	<ul>
	{% for question in latest_question_list %}
	# We don't use specific URL Paths for the case that we need to update it. See urlpatterns where we declare the name of the url to go to via the name 'detail`
	<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li></a></li>
	{% endfor %}
	</ul>
{% else %}
	<p>No polls are available.</p>
{% endif %}
```



4. Update  `views.py` to use the template: 

Method A: Using `loader` for template
```
from django.template import loader

def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5]
	template = loader.get_template('polls/index.html')
	context = {
	'latest_question_list': latest_question_list,
	}
	return HttpResponse(template.render(context, request))
```
Method B: Using ` render` for template
```
from django.shortcuts import render

def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5]
	context = {'latest_question_list': latest_question_list}
	return render(request, 'polls/index.html', context)
```
