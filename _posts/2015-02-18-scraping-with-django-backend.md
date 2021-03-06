---
layout: post
title: Scraping with Django Backend
---

I like to use [Django](https://www.djangoproject.com/) as a backend in my scraping scripts. 
I use it to model the data being scraped with the [Django ORM](https://docs.djangoproject.com/en/1.7/topics/db/models/).
Then, once the data has been collected, I view the results using the [automatic admin 
interface](https://docs.djangoproject.com/en/1.7/intro/tutorial02/) which is easy to 
setup in just a few lines of code.

In this post, I'll show how to set up a project template for scraping with Django. 

First let's set up the environment. Create a directory for the template and start up a
virtual environment in that directory.

```bash
$ mkdir django_scraping_template && cd django_scraping_template
$ virtualenv venv
$ source venv/bin/activate
```

Then install the dependencies (Django obviously being one of them) that you want to use
for your scraping work:

```bash
$ pip install mechanize
$ pip install beautifulsoup4
$ pip install Django
$ pip freeze > requirements.txt
```

Next create the Django project for the settings and an app for modeling the data:

```bash
$ django-admin.py startproject scraper
$ cd scraper
$ python manage.py startapp custom_scraper
```

In the top level directory, create a file named scraper.py. We'll edit that it in a 
second. 

Your directory layout at this point should look as follows:

```bash
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
```

Edit the scraper/settings.py file, and change the INSTALLED\_APPS setting to include the string 'custom_scraper'. 
So it’ll look like this:

```python
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'custom_scraper'
)
```

Edit custom_scraper/models.py and create your model definitions. Then run migrate
to sync the models into the database.

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

Now in the top scraper directory edit the scraper.py file:

```python
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
```

The model(s) you define in custom_scraper/models.py can now be used in this standalone script.

Now just make the script executable and you're all set.

```bash
$ chmod +x scraper.py
$ ./scraper.py
```

## Shameless Plug

Have a scraping project you'd like done? I'm available for hire. [Contact me](/contact) 
with some details about your project and I'll give you a quote.

