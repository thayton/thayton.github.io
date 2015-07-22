#include <stdio.h>

int fn1(char *s) { return 1; }
int fn2(char *s) { return 2; }

typedef int (*fnptr)(char *); /* typedef makes it easier to read */

int main(int argc, char **argv)
{
  int (*ptr[])(char *) = { fn1, fn2 }; /* w/o typedef */
  fnptr a[] = { fn1, fn2 }; /* with typdef */

  printf("%d\n", ptr[0]("one"));
  printf("%d\n", ptr[1]("two"));

  printf("%d\n", a[0]("one"));
  printf("%d\n", a[1]("two"));

  return 0;
}
