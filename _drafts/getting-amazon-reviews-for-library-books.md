---
layout: post
title: Getting Amazon Reviews for Library Books with Python
---

Often when I want to read about a new technical subject, I'll see what's available for free at the 
local library before buying a new book. To decide which book to check out, I browse through the library's
catalogue and look up the reviews for each book on Amazon to see which book is the best rated. 
Of course, doing this manually is tedious and repetitive which is usually a sign that it should be 
automated. 

In this post I'll go over a script I developed that queries a library catalogue for books on a particular 
subject and then returns the books sorted according to their Amazon rating.

## Overview

My local library uses [SirsiDynix](http://www.sirsidynix.com/) for its catalogue software. Here's a 
screenshot of their advanced search page:

![Search Image](/assets/sirsi/advanced_search.png)

As you can see, searches can be filtered by format type (Electronic Resources, Sound Recording, Books, etc.) 
For our purposes we only want books. Once you submit the search, you get a results page like the following:

![Results Image](/assets/sirsi/search_results.png)

The URL for searches limited to 'Books' has the following format:

https://mdpl.ent.sirsi.net/client/catalog/search/results?qu=javascript&qf=FORMAT%09Format%09BOOK%09Books

So, to query for a particular subject, we just need to set the `qu` parameter in the URL and then scrape 
the results. Since results are sorted by relevance we'll limit our scrape to the first page of results. 

What part of the results do we need to scrape? Well, we'd like the title obviously, and also the book's 
ISBN number so we can look it up on Amazon to get its rating.

Here's a summary of everything we'll need the script to do:

* Query the Sirsi catalogue for books on a particular subject
* Scrape the book title and ISBN number for the first page of results
* Use the ISBN number to get the Amazon page for each book
* Scrape the rating for each book from its Amazon page
* Sort the books in reverse order by their Amazon rating

## Implementation

A top level `scrape()` method encapsulates all of our logic.

{% highlight python %}
def scrape(self, q):
   books = self.search_library_books(q=q)
   self.get_amazon_reviews(books)
   books = self.rank_by_reviews(books)

   for b in books:
       print b['rating'], b['title']
{% endhighlight %}

### Scraping the Library Books

To scrape the book titles and ISBN numbers from the Sirsi catalogue, we filter based on the 
`class` and `id` attributes of the elements containing the title and ISBN number. 

{% highlight python %}
def search_library_books(self, q):
    '''
    Scrape the first page of books for the given query.
    Calculate the ISBN10 value for each book so we can 
    look it up on Amazon
    '''
    books = []

    self.br.open(LIBRARY_URL.format(q))

    s = BeautifulSoup(self.br.response().read())
    x = {'class': 'isbnValue'}
    y = re.compile(r'^results_cell\d+$')
    z = re.compile(r'^results_bio\d+$')

    for i in s.findAll('input', attrs=x):
        d = i.findParent('div', id=y)
        d = d.find('div', id=z)

        if not i.get('value'):
            continue

        book = {}
        book['title'] = d.a['title']
        book['isbn13'] = i['value']
        book['isbn10'] = self.isbn13to10(book['isbn13'])
        books.append(book)

    return books
{% endhighlight %}

### Amazon Page Links

Note that we scrape the ISBN13 number and then convert it to ISBN10. That's because while the 
Sirsi catalogue uses ISBN13 numbers, Amazon uses ISBN10 numbers for product links. [1][2]

### ISBN13 to ISBN10 Conversion
[Wikipedia](http://en.wikipedia.org/wiki/International_Standard_Book_Number#ISBN-10_check_digit_calculation) has a nice writeup
on how ISBN numbers are formatted. To convert an ISBN13 number to ISBN10, we chop off the first 3 digits of the ISBN13 number, then
calculate the ISBN10 check digit for the next nine numbers. The nine numbers plus the check digit is the ISBN10 number.

{% highlight python %}
def isbn13to10(self, isbn13):
    '''
    Convert an ISBN13 number to ISBN10 by chomping off the
    first 3 digits, then calculating the ISBN10 check digit 
    for the next nine digits to come up with the ISBN10 
    '''
    first9 = isbn13[3:][:9]
    isbn10 = first9 + self.isbn10_check_digit(first9)
    return isbn10
{% endhighlight %}

The formula for calculating an ISBN10 check digit (the 10th number) given the first nine numbers
is as follows:

(11 - ((10x<sub>1</sub> + 9x<sub>2</sub> + 8x<sub>3</sub> + 7x<sub>4</sub> + 6x<sub>5</sub> + 5x<sub>6</sub> + 4x<sub>7</sub> + 3x<sub>8</sub> + 2x<sub>9</sub>) mod 11)) mod 11

{% highlight python %}
def isbn10_check_digit(self, isbn10):
    '''
    Given the first 9 digits of an ISBN10 number calculate
    the final (10th) check digit
    '''
    i = 0
    s = 0

    for n in xrange(10,1,-1):
        s += n * int(isbn10[i])
        i += 1

    s = s % 11
    s = 11 - s
    v = s % 11
        
    if v == 10:
        return 'x'
    else:
        return str(v)
{% endhighlight %}

### Amazon Reviews

Once we've got the ISBN10 number for the book, we can look it up on Amazon using the following link
format:

http://www.amazon.com/gp/product/\<ISBN10\>

Then we can go to that page and scrape the rating for the book.

{% highlight python %}
def get_amazon_reviews(self, books):
    '''
    Get the Amazon review/rating for each book in books[]
    '''
    for b in books:
        url = AMAZON_URL + b['isbn10']

        self.br.open(url)

        s = BeautifulSoup(self.br.response().read())
        d = s.find('div', id='avgRating')
        m = re.search(r'\d+(\.\d+)?', d.text.strip())
        f = float(m.group(0))

        b['rating'] = f
{% endhighlight %}

### Sorting the Results

Once we've got all the rating for all of the books, we sort them according to their rating.

{% highlight python %}
def rank_by_reviews(self, books):
    ''' 
    Sort books by rating with highest rated books first
    '''
    newlist = sorted(books, key=lambda k: k['rating'], reverse=True) 
    return newlist
{% endhighlight %}

## Shameless Plug

Have a scraping project you'd like done? I'm available for hire. [Contact me](/contact) 
for a free quote.

[1] http://www.newselfpublishing.com/AmazonLinking.html 
[2] https://affiliate-program.amazon.com/gp/associates/help/t5/a16