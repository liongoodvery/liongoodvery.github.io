---
layout: post
title: Linux Memory Management
comments: true
date: 2016-10-15 17:42:32+00:00
categories:
- Linux
- Tech
tags:
- linux
- memory
main-class: 'Linux'
color: '#006798'

introduction: 'Linux Memory Management'

---
## Linux Memory Management

### The Process Address Space

#### Pages And Paging

#### Allocating Dynamic Memory

```c
#include <stdlib.h>

void * malloc (size_t size);
```

- The return value and its type


- A simple example

```c

#include<stdio.h>
#include<stdlib.h>
int main(){
    char* p;
    p=malloc(2048);
    if(!p){
        perror("malloc failed");
    }
}
```

- A common wrap

```c
void* xmalloc(size_t size){
    void* p;
    p=malloc(size);
    if(!p){
        perror("malloc failed");
        exit(EXIT_FAILURE);
    }
    return p;
}
```

- Allocating Arrays

```c
#include <stdlib.h>

void * calloc (size_t nr, size_t size);
```

```c
void* xcalloc(size_t nr,size_t size){
    void* p;
    p=calloc(nr,size);
    if(!p){
        perror("calloc failed");
        exit(EXIT_FAILURE);
    }
    return p;

```
-  zeros all bytes in the returned chunk of memory


```c
int main(){
    int *p,*q;
    p=(int*)xmalloc(50*sizeof(int));
    q=(int*)xcalloc(50,sizeof(int));
    printf("%d\n",q[0]);
    return 0;
```

- Resizing Allocations


```c
#include<stdio.h>
#include<stdlib.h>
#include "malloc.h"
int main(){
    int *p;
    p=xcalloc(2,sizeof(int));
    printf("size of p %d\n",sizeof(p));
    p=realloc(p,sizeof(int));
    printf("size of p %d\n",sizeof(p));
    //dangerous
    for(int i = 10;i<1<<10;++i){
        p[i] = 1;
    }
}
```

- Freeing Dynamic Memory

```c
#include <stdlib.h>

void free (void *ptr);
```



```c
#include<stdio.h>
#include<stdlib.h>

void print_chars(int n,char c){
    int i;
    for(i=0;i<n;++i){
        char *s;
        s=(char*)calloc(i+2,1);
        int j;
        for(j=0;j<i+1;++j){
            s[j]=c;
        }
        printf("%s\n",s);
        free(s);
    }
}

int main(){
    print_chars(10,'x');
    return 0;
}
```


result:

```
x
xx
xxx
xxxx
xxxxx
xxxxxx
xxxxxxx
xxxxxxxx
xxxxxxxxx
xxxxxxxxxx
```
#### Alignment


#### Managing the Data Segment
