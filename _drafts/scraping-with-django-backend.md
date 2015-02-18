---
layout: post
title: Scraping with Django Backend
---

I like to use [Django](https://www.djangoproject.com/) as a backend in my scraping scripts. 
I use it to model the data being scraped with the [Django ORM](https://docs.djangoproject.com/en/1.7/topics/db/models/).
Then, once the data has been collected, I view the results using the [automatic admin 
interface](https://docs.djangoproject.com/en/1.7/intro/tutorial02/) which is easy to 
setup in just a few lines of code.

In this post, I'll show how to set up a template for scraping projects with Django. 

First let's set up the environment. Create a directory for the template and start up a
virtual environment in that directory.

{% highlight bash %}
$ mkdir django_scraping_template && cd django_scraping_template
$ virtualenv venv
$ source venv/bin/activate
{% endhighlight %}

Then install the dependencies (Django obviously being one of them) that you want to use
for your scraping work:

{% highlight bash %}
$ pip install mechanize
$ pip install beautifulsoup4
$ pip install Django
$ pip freeze > requirements.txt
{% endhighlight %}

Next create the Django project for the settings and an app for modeling the data:

{% highlight bash %}
$ django-admin.py startproject scraper
$ cd scraper
$ python manage.py startapp custom_scraper
{% endhighlight %}

In the top level directory, create a file named `scraper.py`. We'll edit that it in a 
second. 

Your directory layout at this point should look as follows:

{% highlight bash %}
django_scraping_template
├── scraper.py
└── scraper
    ├── scraper
    │   ├── wsgi.py
    │   ├── urls.py
    │   ├── settings.py
    │   └── __init__.py
    ├── manage.py
    └── custom_scraper
        ├── views.py
        ├── tests.py
        ├── models.py
        ├── migrations
        │   └── __init__.py
        ├── admin.py
        └── __init__.py
{% endhighlight %}

Edit the scraper/settings.py file, and change the INSTALLED\_APPS setting to include the string 'custom_scraper'. 
So it’ll look like this:

{% highlight python %}
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'custom_scraper'
)
{% endhighlight %}

Edit customer_scraper/models.py and create your model definitions. Then run migrate
to sync the models into the database.

{% highlight bash %}
$ python manage.py makemigrations
$ python manage.py migrate
{% endhighlight %}

Now in the top scraper directory edit the `scraper.py` file:

{% highlight python %}
#!/usr/bin/env python                                                                                                                                                                
import os
import sys
import django

sys.path.append(os.path.realpath(os.path.join(os.path.dirname(__file__), 'scraper/')))
sys.path.append(os.path.realpath(os.path.join(os.path.dirname(__file__), 'scraper/scraper/')))

os.environ['DJANGO_SETTINGS_MODULE'] = 'settings'
django.setup()

from django.core.exceptions import ObjectDoesNotExist
from custom_scraper.models import *

class CustomScraper(object):
    def __init__(self):
        self.url = ""

    def scrape(self):
        print 'scraping...'

if __name__ == '__main__':
    scraper = CustomScraper()
    scraper.scrape()
{% endhighlight %}

The model(s) you created in models.py can be used in this standalone script for the data
you scrape.

Now just make the script executable and you're all set up.

{% highlight bash %}
$ chmod +x scraper.py
$ ./scraper.py
{% endhighlight %}


