http://www.yolinux.com/TUTORIALS/LibraryArchives-StaticAndDynamic.html

```c
#include <stdio.h>

void hello(void);

void
hello(void)
{
    printf("hello\n");
}
```

gcc -Wall -fPIC -c hello.c
gcc -shared -Wl -o libhello.so hello.o 

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int main(int argc, char **argv)
{
    void *lib_handle;
    void (*fn)(void);
    int x;
    char *error;

    lib_handle = dlopen("libhello.so", RTLD_LAZY);
    if (!lib_handle) 
        {
            fprintf(stderr, "%s\n", dlerror());
            exit(1);
        }

    printf("loaded into memory @ %p\n", lib_handle);

    fn = dlsym(lib_handle, "hello");
    if ((error = dlerror()) != NULL)  
        {
            fprintf(stderr, "%s\n", error);
            exit(1);
        }

    (*fn)();

    dlclose(lib_handle);
    return 0;
}
```

gcc -rdynamic -o loadhello loadhello.c -ldl

```
$ ./loadhello 
loaded into memory @ 0x7fc9c0403970
hello
```


