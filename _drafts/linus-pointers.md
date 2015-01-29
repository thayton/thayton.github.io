http://grisha.org/blog/2013/04/02/linus-on-understanding-pointers/

#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

struct queue_node {
  int val;
  struct queue_node *next;
};

static void enqueue(struct queue_node **queue, int val);
static int dequeue(struct queue_node **queue, int *val);

int main(int argc, char **argv)
{
  struct queue_node *q = NULL;
  int v;

  enqueue(&q, 1);
  enqueue(&q, 2);
  enqueue(&q, 3);
  
  while (dequeue(&q, &v))
    printf("%d ", v);

  printf("\n");
  exit(0);
}

static void
enqueue(struct queue_node **queue, int val)
{
  struct queue_node **ppn;
  struct queue_node  *pn;

  for (ppn = queue, pn = *ppn; pn; pn = *ppn)
    ppn = &pn->next;

  if ( (pn = malloc(sizeof *pn)) == NULL) {
    perror("malloc");
    exit(1);
  }

  pn->val = val;
  pn->next = NULL;
  *ppn = pn;
}
