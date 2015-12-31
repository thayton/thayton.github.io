---
layout: post
title: "Book Review: Web Scraping with Python"
---

I just finished reading <a target="_blank" href="">Web Scraping with Python</a> by Richard Lawson.

The book is great and manages to cover a lot of different scraping topics in just 140 pages. There
is just enough information on each topic to get you going without getting bogged down on any one topic.

The book starts out with four chapters dedicated to building a web scraping framework. 

Start with simple caching on the disk. Next goes
on to show how to implement caching using MongoDB. The code was written to be pluggable and reusable
which I liked. The author shows just enough code to illustrate an idea so you don't get bogged down
in details. 

The next three chapters cover dynamic websites, forms and logins, and CAPTCHAs. The chapter on dynamic 
websites shows how to reverse engineer websites that using AJAX on the backend. Then it goes on to cover 
how to use either Selenium to drive a browser or GWT in order to handle dynamic websites by letting the
javascript render the pages.

The chapter on forms is very useful. After showing the basics of form submission with urllib, the
author provides some neat details on how you can reuse the cookies from your Firefox browsing sessions 
to log into a site within your scraping script. 

The CAPTCHA chapter starts off bying showing how to use Optical Character Recognition (OCR) to solve
some basic CAPTCHAs using thresholding. For more complicated CAPTCHAs, the author shows how a popular
(and free) CAPTHCA solving service can be used from within a script.

Finally the wraps things up with a chapter on basics of the popular Python scraping framework Scrapy (and 
its visual counterpart Portia) and a final chapter showing how to scrape some popular websites like 
Gap.

