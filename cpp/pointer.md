## 1. 指针、数组和字符串

### 代码

```c++
#include <iostream>

void increment(int *a) {
  *a += 1;
}

void print_str(char a[]) {
  while (*a != '\0') {
    printf("%c", *a++);
  }
  printf("\n");
}

void update_str(char a[]) {
  while (*a != '\0') {
    *a++ += 1;
  }
}

int main() {
  int a = 10;
  int *p;
  p = &a; // 获取地址称为引用(reference)。&a 读作 ampersand a
  printf("%d\n", *p); // 通过地址取值称为取消引用(dereference)。*p 读作 asterisk p

  // 为什么指针要使用强类型(strong type)，而不是通用类型(generic type)？
  // 因为对于无论是基本数据类型还是自定义数据类型，每种数据类型所占字节不同、且二进制的存储规则不同

  // 如果定义通用类型的指针，它不能 取消引用
  void *c = p;

  // pointers to pointers
  int **p2 = &p;
  int ***p3 = &p2;
  printf("%d\n", ***p3); // 10

  // call by reference
  increment(p);
  printf("%d\n", *p); // 11

  // 数组名会返回一个指向数组第一项的地址
  int arr[5] = {11, 22, 33, 44, 55};
  printf("%d\n", *(arr + 1)); // 等价于访问 arr[1]

  // arrays as function arguments
  // 获取数组的长度
  int len = sizeof(arr) / sizeof(arr[0]);

  // 字符串就是字符类型的数组，但是要以 \0 作为字符串结尾。
  char str[6] = "Hanna";
  char str2[6] = {'H', 'a', 'n', 'n', 'a', '\0'};
  // pass by reference
  print_str(str); // "Hanna"
  update_str(str2);
  print_str(str2); // "Iboob"

  // 不要这样定义字符串！这是声明一个指针指向一个常量字符串
  char *st = "hello";
  // 这是错误的，常量字符串无法被修改
  st[3] = 'd';
}
```

## 2. 多维数组

### 二维数组

```c++
#include <iostream>

using namespace std;

void do_nothing(int (*a)[3]); // 或者写为 a[][3]

int main() {
  int a[2][3] = {
          {11, 22, 33},
          {99, 88, 77}
  };

  // 二维数组的存储也是顺序的，所以可以直接通过 a[0][x] 来访问
  for (int i = 0; i < 6; ++i) {
    cout << a[0][i] << ' '; // 11 22 33 99 88 77
  }
  cout << endl;

  // a
  cout << a << endl; // 0x7ffeea88d790，a 返回的是一维数组的地址
  int (*p0)[3] = a; // 等价于 =&a[0]
  cout << "the value of p0 is " << *p0 << endl; // the value of p0 is 0x7ffee9e8d790
  cout << **p0 << endl; // 11
  cout << *(*p0 + 2) << endl; // 33，等价于 p0[0][2]
  cout << (*p0)[4] << endl; // 88，等价于 p0[0][4] 或 a[0][4]
  cout << (*(p0 + 1))[2] << endl; // 77，等价于 p0[1][2]

  // a[0]
  cout << a[0] << endl; // 0x7ffeea88d790，和 a 的值相同，但它返回的是数组首项的地址
  int *p1 = a[0]; // 等价于 =&a[0][0]
  for (int i = 0; i < 3; ++i) {
    cout << *p1++ << ' '; // 11 22 33
  }
  cout << endl;

  // a[1]
  cout << a[1] << endl; // 0x7ffeea88d79c,比 a[0] 多 12
  int *p2 = a[1];
  for (int i = 0; i < 3; ++i) {
    cout << *p2++; // 99 88 77
  }
  cout << endl;
  
  // 作为函数参数
  do_nothing(a);
}
```

### Pointers and multi-dimensional arrays

多维数组和二维是一样的。

```c++
#include <iostream>

void func(int (*a)[3][4]) {} // 行参：或者是 a[][3][4]

int main() {
  int a[2][3][4] = {
          {
                  {11,  22,  33,  44},
                  {2,  3,  4,  5},
                  {9,  8,  7,  6}
          },
          {
                  {911, 922, 933, 944},
                  {92, 93, 94, 95},
                  {99, 98, 97, 96}
          }
  };
  int (*p)[3][4] = a;

  printf("%d\n", *(*(*p + 1) + 2)); // 4，等价于 p[0][1][2]
  printf("%d", *(**p + 6)); // 4，等价于p[0][0][6]

  func(a);
}
```

## 3. 动态内存

c++程序内存包括：栈(Stack)、堆(Heap)、全局变量(Static/Global)、代码(Code Text)。

> 程序内存中的堆和栈，并不是数据结构中的堆和栈。
>
> 程序内存里的栈是栈数据结构的实现，但内存里的堆不是堆数据结构的实现。

**栈**用于函数的调用执行。在程序执行时，堆的大小不会改变，所以函数嵌套运行到一定量级时，会导致内存溢出。

**堆**也称为动态内存池(free pool/store of memory)，它用于动态内存分配。它没有空间大小的限制。

#### 代码

在C中使用`malloc`、`calloc`、`realloc`和`free`。

在C++中使用`new`和`delete`。当然也可以使用C的关键字。

C代码：

```c++
#include <stdio.h>
#include <stdlib.h>

int main() {
  int a; // goes on stack
  int *p; // goes on stack

  // malloc 在堆上分配一个指定大小的空间，它返回一个 void* 类型，需要转为需要的类型
  p = (int *) malloc(sizeof(int)); // 将 p 指向堆的一个地址
  *p = 10;
  // 释放空间
  free(p);

  // 分配新的空间
  p = (int *) malloc(20 * sizeof(int));
  *(p + 5) = 99;
  printf("%d", p[5]); // 99
  free(p);
}
```

C++代码：

```c++
#include <stdio.h>

int main() {
  int a; // goes on stack
  int *p; // goes on stack

  p = new int;
  *p = 10;
  delete p;

  p = new int[20];
  *(p + 5) = 99;
  printf("%d", p[5]); // 99
  delete[] p;
}
```

#### malloc、calloc和realloc

```c++
#include <stdio.h>
#include <stdlib.h>

int main() {
  int n;
  printf("Enter size of array\n");
  scanf("%d", &n);

  // 创建数组，由于 n 是变量，所以不能使用 int a[n] = {};

  // 使用动态分配内存 malloc
  int *ar = (int *) malloc(n * sizeof(int));
  for (int i = 0; i < n; ++i) {
    ar[i] = i + 91;
  }

  for (int i = 0; i < n; ++i) {
    printf("%d", ar[i]);
    if (i != n - 1) printf(", ");
  }
  printf("\n");

  // 使用 calloc
  // 和 malloc 的区别：
  /// 1. 参数写法不同
  /// 2. calloc 会自动对值进行初始化，而 malloc 生成的默认是垃圾值
  int *ar2 = (int *) calloc(n, sizeof(int));
  free(ar2);

  // realloc: 重新调整之前使用 malloc / calloc 生成的内存块的大小
  ar = (int *) realloc(ar, n / 2 * sizeof(int)); // 缩小空间
  
  int *ar3 = (int *) calloc(2, sizeof(int)); // 重新请求内存块
  ar3[0] = 888;
  
  for (int i = 0; i < n; ++i) {
    printf("%d", ar[i]); // 这儿输出结果可能不会有变化，取决于机器
    if (i != n - 1) printf(", ");
  }
}
```

### Pointers as function returns

指针如果作为函数的返回值，如果指针是在函数中定义的，则必须在堆中开辟空间。因为函数执行完毕后，局部变量所在的函数占用的栈空间会自动释放。

```c++
#include "stdio.h"

int *add(int *a, int *b) {
  int c = *a + *b;
  int *ret = new int;
  *ret = c;
  return ret;
}

void say_hello() {
  printf("Hello world\n");
}

int main() {
  int a = 3;
  int b = 4;
  int *c = add(&a, &b);
  say_hello();
  printf("c is %d", *c);
}
```

## 4. 函数指针和回调

### 函数指针

代码：

```c++
#include "stdio.h"

int add(int a, int b) {
  return a + b;
}

void say_hello() {
  printf("Hello world\n");
}

int main() {
  // 最简单的赋值形式就是直接使用函数名称，然后使用时直接使用指针名称
  // 括号不能去掉哦！
  int (*p)(int, int) = add; // 或者 =&add

  void (*p2)() = &say_hello; // 或者 =say_hello

  int c = p(1, 2); // 或者 (*p)(1, 2)
  p2(); // 或者 (*p2)()

  printf("%d", c);
  return 0;
}
```

### 回调

这里的示例是：给冒泡排序传递一个排序方法。

```c++
#include "stdio.h"

int bubble_sort(int ar[], int n, int (*compare)(int, int)) {
  for (int i = 0; i < n - 1; ++i) {
    for (int j = n - 1; j > i; --j) {
      if (compare(ar[j - 1], ar[j]) > 0) {
        int temp = ar[j - 1];
        ar[j - 1] = ar[j];
        ar[j] = temp;
      }
    }
  }
}

int low_to_high(int a, int b) {
  if (a > b) return 1;
  else return -1;
}

int high_to_low(int a, int b) {
  if (a > b) return -1;
  else return 1;
}

int main() {
  int n = 6;
  int ar[] = {324, 12, 8, 1234, 99, -111};
  int (*p)(int, int) = low_to_high; // 1. 使用函数指针
  bubble_sort(ar, n, p);
  for (int i = 0; i < n; ++i) {
    printf("%d ", ar[i]); // -111 8 12 99 324 1234 
  }
  printf("\n---------\n");

  bubble_sort(ar, n, high_to_low); // 2. 函数名可以直接作为参数
  for (int i = 0; i < n; ++i) {
    printf("%d ", ar[i]); // 1234 324 99 12 8 -111 
  }
}
```

使用stdlib自带的qsort：

```c++
#include "stdio.h"
#include "stdlib.h"

int func(const void *a, const void *b) {
  int A = *((int *) a);
  int B = *((int *) b);
  return A > B;
}

int main() {
  int n = 6;
  int ar[] = {324, 12, 8, 1234, 99, -111};
//  int (*compare)(void *, void *) = func;
  // stdlib 中的快速排序
  qsort(ar, n, sizeof(int), func);

  for (int i = 0; i < n; ++i) {
    printf("%d ", ar[i]); // -111 8 12 99 324 1234 
  }
}
```

### 内存泄漏(memory leak)

**栈**中的内存是程序自动管理的，函数执行完毕会自动释放所在的栈内存。

**堆**是程序员手动控制的存储空间，如果使用完毕的变量不释放的话，它将在程序执行期间一直占用内存，导致内存泄漏。所以一定要使用`free/delete`来释放动态内存。