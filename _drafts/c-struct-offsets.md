#include <stdio.h>

#define OFFSETOF(s,f) (long) &(((struct s *)0)->f)

struct queue_node {
  char *key;
  int val;
  char pad;
  struct queue_node *next;
};

int main(int argc, char **argv)
{
  printf("%ld\n", OFFSETOF(queue_node, key));
  printf("%ld\n", OFFSETOF(queue_node, val));
  printf("%ld\n", OFFSETOF(queue_node, pad));
  printf("%ld\n", OFFSETOF(queue_node, next));
  return 0;
}


