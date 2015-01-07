Explain why you have char payload[1] at end of struct:

struct some_struct {
   field1
   field2
   ...
   char payload[1];
}

Explain why you use an array here and why you can't use a pointer.