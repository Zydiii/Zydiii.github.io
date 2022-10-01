---
layout: post
title: "isdigit"
date: 2022-10-01
feature_image: images/2022.8.21/0.png 
tags: [C++]
---

<!--more-->

## isdigit

可以用来判断字符是否为数字，即是否为 "0, 1, 2, 3, 4, 5, 6, 7, 8, 9"

```c++
/* isdigit example */
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
int main ()
{
  char str[]="1776ad";
  int year;
  if (isdigit(str[0]))
  {
    year = atoi (str);
    printf ("The year that followed %d was %d.\n",year,year+1);
  }
  return 0;
}
```