Given the following function to reverse a string of length n:

{% highlight c %}
static char *
revs(char *s, size_t n)
{
  char *l, *r, t;

  for (l = s, r = s + n - 1; l < r; l++, r--) {
    t = *l;
    *l = *r;
    *r = t;
  }

  return s;
}
{% endhighlight %}

Why do we need to declare two separate character arrays in the example below?

{% highlight c %}
#include <stdio.h>
#include <string.h>

static char *revs(char *s, size_t n);

int main(int argc, char **argv)
{
  char x[] = "the sheltering sky";
  char y[] = "the sheltering sky";

  printf("revs(%s) => %s\n", x, revs(y, strlen(y)));

  return 0;
}
{% endhighlight %}


Would `printf("revs(%s) => %s\n", x, revs(x, strlen(x)));` have worked?

What about if x and y were declared as pointers: 

{% highlight c %}
char *x = "the sheltering sky";
char *y = "the sheltering sky";
{% endhighlight %}

