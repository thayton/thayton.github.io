---
layout: post
title: Scraping with Django Backend
---

{% highlight bash %}
$ mkdir scraper && cd scraper
$ virtualenv venv
$ source venv/bin/activate
{% endhighlight %}

{% highlight bash %}
$ pip install mechanize
$ pip install beautifulsoup4
$ pip install Django
$ pip freeze > requirements.txt
{% endhighlight %}

{% highlight bash %}
$ django-admin.py startproject scraper
$ cd scraper
$ python manage.py startapp crossfit_scraper
{% endhighlight %}

Edit the scraper/settings.py file, and change the INSTALLED\_APPS setting to include the string 'crossfit_scraper'. 
So it’ll look like this:

{% highlight python %}
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'crossfit_scraper'
)
{% endhighlight %}

{% highlight bash %}
$ python manage.py makemigrations
$ python manage.py migrate
{% endhighlight %}

{% highlight bash %}

{% endhighlight %}

scraper.py

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
from crossfit_scraper.models import *

class CrossfitScraper(object):
    def __init__(self):
        self.url = "http://map.crossfit.com/affinfo.php?a={}&t=0"

    def scrape(self):
        print 'scraping...'

if __name__ == '__main__':
    scraper = CrossfitScraper()
    scraper.scrape()
{% endhighlight %}
