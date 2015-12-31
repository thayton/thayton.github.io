---
layout: post
title: "Book Review: Web Scraping with Python"
---

I just finished reading <a target="_blank" href="http://www.amazon.com/Web-Scraping-Python-Richard-Lawson/dp/1782164367/">
Web Scraping with Python</a> by Richard Lawson; Packt Publishing.

<img style="width: 50%; height: 50%; margin:0 15px 15px 0" src="/assets/book_review_wswp/wswp_cover.jpg">

The book is terrific and manages to cover a lot of important scraping topics in just 140 pages. The
<a target="_blank" href="https://www.packtpub.com/books/info/authors/richard-penman">author</a> provides 
enough information so that by the end of the book you've got an arsenal of techniques and code for scraping 
a variety of websites. At the same time, the book avoids getting bogged down on any one topic.

### Chapters 1-4

The book starts out with four chapters dedicated to building a basic web scraping framework. Starting 
from a simple link crawler, the author progressively adds more and more capabilities so that by the end 
of chapter four you've got a web scraping framework that provides:

- Randomized Proxy support
- Caching: Shown first with simple caching on the disk. Then using MongoDB. 
- Parralellism: Shown first with threads. Then via concurrent processes.
- HTTP Error handling

During the course of developing these features, the book also shows useful performance comparisons between
HTML parsing solutions (regex, BeautifulSoup, and lxml) and parallelization techniques (Python threads and
multiprocessing).

The <a target="_blank" href="https://bitbucket.org/wswp/code">source code</a> is written to be pluggable 
and reusable which I liked. The author shows just enough code to illustrate an idea without making you
wade through pages of code.

### Chapters 5-7

The next three chapters go into the nuts and bolts of scraping dynamic websites, handling forms and logins, 
and dealing with CAPTCHAs. 

The chapter on dynamic websites shows how <a target="_blank" href="https://getfirebug.com/firebuglite">FireBug Lite</a> 
can be used to reverse engineer websites using AJAX on the backend. Then it delves into different options (WebKit, 
Selenium, ...) for rendering dynamic websites.

Next comes a chapter dedicated to handling forms. After showing the basics of form submission with urllib
and Mechanize the author provides useful details on how you can reuse the cookies from your Firefox browsing 
sessions to log into a site within your scraping script. 

The CAPTCHA chapter starts off bying showing how to use Optical Character Recognition (OCR) to solve
some basic CAPTCHAs using thresholding. For more complicated CAPTCHAs, the author shows how a popular
(and free) CAPTCHA solving service can be used from within a script.

### Chapters 8-9

Finally the book wraps things up with a chapter on the popular Python scraping framework 
<a target="_blank" href="http://scrapy.org/">Scrapy</a> (and its visual counterpart 
<a target="_blank" href="https://github.com/scrapinghub/portia">Portia</a>) and a final chapter 
that provides real life examples of how to scrape some popular websites like Gap.

### Conclusion

All in all a great resource for both beginners and experienced web scrapers alike. If you're just
starting out, this book will get you up to speed quickly. For experienced scrapers, you'll likely 
pick up a few tips you can add to your scraping toolbelt.