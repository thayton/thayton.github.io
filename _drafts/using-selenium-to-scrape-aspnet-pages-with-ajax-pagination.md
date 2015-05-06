---
layout: post
title: Using Selenium to Scrape ASP.NET Pages with AJAX Pagination
---

In my [last post]({% post_url 2015-05-04-scraping-aspnet-pages-with-ajax-pagination %}) I went
over the nitty-gritty details of how to scrape an ASP.NET AJAX page using Python [mechanize](http://wwwsearch.sourceforge.net/mechanize/). 
Since mechanize can't process Javascript, we had to understand the underlying data formats used 
when sending form submissions, parsing the server's response, and how pagination is handled.
In this post, I'll show how much easier it is to scrape the exact same site when we use Selenium
in conjunction with PhantomJS.

## Background

The site I'll use in this post is the same as the one I used in my last post, the search form 
provided by [The American Institute of Architects](http://www.aia.org/) for finding architecture 
firms in the US. 

[http://architectfinder.aia.org/frmSearch.aspx](http://architectfinder.aia.org/frmSearch.aspx)

Once again, I'll show to scrape the names and links of the firms listed for all of the states
in the form.