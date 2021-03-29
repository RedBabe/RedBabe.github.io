## 1. 单链表

### Singly Linked List

插入、删除和反转：

```c++
#include "stdio.h"
#include "assert.h"
#include <stack>

struct Node {
  int data;
  struct Node *next;
};

Node *insert(Node *head, int x, int pos) {
  assert(pos >= 0);
  Node *temp = new Node{
          x, NULL
  };
  if (pos == 0) {
    temp->next = head;
    return temp;
  }

  Node *front = head;
  for (int i = 0; i < pos - 1; ++i) {
    assert(front != NULL && front->next != NULL);
    front = front->next;
  }
  temp->next = front->next;
  front->next = temp;
  return head;
}

Node *remove(Node *head, int pos) {
  assert(head != NULL && pos >= 0);
  Node *del = head;
  if (pos == 0) {
    head = head->next;
    delete del;
    return head;
  }

  Node *front = head;
  for (int i = 0; i < pos - 1; ++i) {
    front = front->next;
    assert(front->next != NULL);
  }
  del = front->next;
  front->next = del->next;
  delete del;
  return head;
}

// print using iteration
void print(Node *head) {
  while (head != NULL) {
    printf(" %d", head->data);
    head = head->next;
  }
  printf("\n");
}

// print using recursion
void print2(Node *head) {
  if (head == NULL) return;
  printf(" %d", head->data);
  print2(head->next);
}

// print using iteration in reverse
void print_r(Node *head) {
  std::stack<int> s;
  while (head != NULL) {
    s.push(head->data);
    head = head->next;
  }
  while (!s.empty()) {
    printf(" %d", s.top());
    s.pop();
  }
}

// print using recursion in reverse
void print2_r(Node *head) {
  if (head == NULL) return;
  print2_r(head->next);
  printf(" %d", head->data);
}

// reverse using iteration
Node *reverse(Node *head) {
  Node *front = NULL;
  Node *rear;
  while (head != NULL) {
    rear = head->next;
    head->next = front;
    front = head;
    head = rear;
  }
  return front;
}


// reverse using recursion
Node *reverse2(Node *head) {
  if (head == NULL || head->next == NULL) return head;
  Node *ret = reverse2(head->next);
  head->next->next = head;
  head->next = NULL;
  return ret;
}

int main() {
  struct Node *head = NULL;

  head = insert(head, 8, 0);
  head = insert(head, 33, 1);
  head = insert(head, 22, 1);
  head = insert(head, 11, 1);
  head = insert(head, 44, 4);
  print(head);
  head = reverse(head);
  print(head);
  head = remove(head, 3);
  print_r(head);
}
```

## 2. 栈

### 栈的简单实现

```c++
#ifndef UNTITLED_STACK_H
#define UNTITLED_STACK_H

template<typename T>
struct Node {
  T data;
  Node<T> *next;
};

// 模版类需要把源代码放到头文件中，否则需要在源代码文件标记：template class Stack<int>;

template<typename T>
class Stack {
public:
  Stack() {
    head = nullptr;
  }

  ~Stack() {
    Node<T> *del;
    while (head != nullptr) {
      del = head;
      head = head->next;
      delete del;
    }
  }

private:
  Node<T> *head;

public:
  void push(T val) {
    Node<T> *temp = new Node<T>{
            val,
            head
    };
    head = temp;
  }

  void pop() {
    if (head == nullptr) return;
    Node<T> *del;
    del = head;
    head = head->next;
    delete del;
  }

  T top() {
    assert(head != nullptr);
    return head->data;
  }

  bool empty() {
    return head == nullptr;
  }
};

#endif //UNTITLED_STACK_H


```

### 判断字符串中的括号是否平衡

```c++
#include <stack>
#include <string>

using namespace std;

bool check_balanced_parentheses(string s) {
  stack<char> st;
  for (char c: s) {
    switch (c) {
      case '{':
      case '(':
      case '[':
        st.push(c);
        break;
      case '}':
        if (st.empty() || st.top() != '{') return false;
        st.pop();
        break;
      case ')':
        if (st.empty() || st.top() != '(') return false;
        st.pop();
        break;
      case ']':
        if (st.empty() || st.top() != '[') return false;
        st.pop();
        break;
      default:
        break;
    }
  }
  return st.empty();
}
```

### 判断括号是否有效

```c++
bool check_balanced_parentheses(string s) {
  stack<char> st;
  for (char c: s) {
    switch (c) {
      case '{':
      case '(':
      case '[':
        st.push(c);
        break;
      case '}':
        if (st.empty() || st.top() != '{') return false;
        st.pop();
        break;
      case ')':
        if (st.empty() || st.top() != '(') return false;
        st.pop();
        break;
      case ']':
        if (st.empty() || st.top() != '[') return false;
        st.pop();
        break;
      default:
        break;
    }
  }
  return st.empty();
}
```

### Infix, Postfix and Prefix Notations

> 中缀符号，后缀符号和前缀符号
>
> 比如`p*2`，`p`是变量(variable)，`2`是常量(constant)，它俩是操作数(operant)，`*`是操作符(operator)，这里也是中缀符号(Infix)。

计算前缀/后缀形式的表达式：

```c++
#include <stack>

// 计算前缀形式的表达式
int calc_prefix(char c[], int len) {
  std::stack<int> s;
  char temp;
  int v1, v2;
  for (int i = len - 2; i >= 0; --i) {
    temp = *(c + i);
    if (temp <= 57 && temp >= 48) {
      s.push(temp - 48);
    } else {
      v1 = s.top();
      s.pop();
      v2 = s.top();
      s.pop();
      switch (temp) {
        case '+':
          s.push(v1 + v2);
          break;
        case '-':
          s.push(v1 - v2);
          break;
        case '*':
          s.push(v1 * v2);
          break;
        case '/':
          s.push(v1 / v2);
          break;
      }
    }
  }
  return s.top();
}

// 计算后缀形式的表达式
int calc_postfix(char c[], int len) {
  std::stack<int> s;
  char temp;
  int v1, v2;
  for (int i = 0; i < len - 1; ++i) {
    // range of number chars is 48 - 57
    temp = *(c + i);
    if (temp <= 57 && temp >= 48) {
      s.push(temp - 48);
    } else {
      v1 = s.top();
      s.pop();
      v2 = s.top();
      s.pop();
      switch (temp) {
        case '+':
          s.push(v2 + v1);
          break;
        case '-':
          s.push(v2 - v1);
          break;
        case '*':
          s.push(v2 * v1);
          break;
        case '/':
          s.push(v2 / v1);
          break;
      }
    }
  }
  return s.top();
}

int main() {
  // 3 * 4 + 5 * 6 - 7 = 35
  char c_prefix[] = "-+*34*567";
  char c_postfix[] = "34*56*+7-";

  printf("%d\n", calc_prefix(c_prefix, sizeof(c_prefix)));
  printf("%d\n", calc_postfix(c_postfix, sizeof(c_postfix)));
  return 0;
}
```

## 3. 队列

### Queue

队列是FIFO结构。比如打印机在打印时，任务进行排队，当前执行的是最先来的打印任务。

和栈不同的是，队列的数组实现，需要将数组需要模拟为环。数组实现略。

链表实现：

```c++
#ifndef UNTITLED_MYQUEUE_H
#define UNTITLED_MYQUEUE_H
#include "assert.h"

template<class T>
struct Node {
  T data;
  Node<T> *next;
};

template<class T>
class MyQueue {
public:
  MyQueue() {
    front = rear = nullptr;
  }

  ~MyQueue() {

  }

private:
  Node<T> *front;
  Node<T> *rear;

public:
  T dequeue() {
    assert(front);
    Node<T> *del = front;
    T ret = del->data;
    front = front->next;
    delete del;
    return ret;
  }

  void enqueue(T val) {
    Node<T> *temp = new Node<T>{
            val,
            nullptr
    };
    if (!front) {
      front = rear = temp;
    } else {
      rear->next = temp;
      rear = rear->next;
    }
  }

  bool empty() {
    return front == nullptr;
  }
};
```

## 4. 树

### Tree

名词：

root: 根节点

parent: 父节点

children: 子节点

sibling: 兄弟节点

grand-parent: 父节点的父节点

leaf: 叶子节点，它没有子节点

sub-tree: 子树。一个叶子结点也可以叫子树

depth / level: 深度 / 级别。根节点的深度为0，其他递增

height: 高度。深度最大的叶子结点其高度为0，其他递增。只有一个节点的树高度为0；空树高度为-1。

树的场景：磁盘存储

### 二分搜索树

> Binary Search Tree (BST)

二分树(Binary Tree)：每一个节点最多只有两个子节点

平衡二分树(Balanced Binary Tree): 所有左子树和右子树的高度差不超过1

二分搜索树：A binary tree in which for each node, value of all the nodes in left subtree is lesser or equal and value of all the nodes in right subtree is greater.

### BST实现

h文件：

```c++
#ifndef UNTITLED_BINARYSEARCHTREE_H
#define UNTITLED_BINARYSEARCHTREE_H

#include <iostream>
#include "assert.h"
#include <queue>

template<class T>
struct BstNode {
  T data;
  BstNode<T> *l;
  BstNode<T> *r;
};

template<class T>
bool isBstUtil(BstNode<T> *node, T mi, T ma) {
  if (node == nullptr) return true;
  if (
          isBstUtil(node->l, mi, node->data)
          && isBstUtil(node->r, node->data, ma)
          && node->data >= mi
          && node->data < ma
          )
    return true;
  return false;
}

// Check if a given binary tree is BST
template<class T>
bool isBst(BstNode<T> *root) {
  return isBstUtil(root, INT_MIN, INT_MAX);
}

template<class T>
class BinarySearchTree {
public:
  BinarySearchTree() {
    root_ptr = nullptr;
  }

private:
  BstNode<T> *root_ptr;

  BstNode<T> *_insert(BstNode<T> *root, BstNode<T> *tmp) {
    if (root == nullptr) return tmp;
    if (tmp->data > root->data) {
      root->r = _insert(root->r, tmp);
    } else {
      root->l = _insert(root->l, tmp);
    }
    return root;
  }

  bool _search(BstNode<T> *root, T data) {
    if (root == nullptr) return false;
    if (root->data == data) return true;
    if (data > root->data) {
      printf("right\n");
      return _search(root->r, data);
    } else {
      printf("left\n");
      return _search(root->l, data);
    }
  }

  void _inorder(BstNode<T> *node) {
    if (node == nullptr) return;
    _inorder(node->l);
    std::cout << node->data << ' ';
    _inorder(node->r);
  }

  void _preorder(BstNode<T> *node) {
    if (node == nullptr) return;
    std::cout << node->data << ' ';
    _preorder(node->l);
    _preorder(node->r);
  }

  void _postorder(BstNode<T> *node) {
    if (node == nullptr) return;
    _preorder(node->l);
    _preorder(node->r);
    std::cout << node->data << ' ';
  }

  BstNode<T> *_find_min(BstNode<T> *node) {
    assert(node);
    BstNode<T> *tmp = node;
    while (tmp->l) {
      tmp = tmp->l;
    }
    return tmp;
  }

  BstNode<T> *_del(BstNode<T> *root, int data, bool &isDeleted) {
    if (root == nullptr) return nullptr;
    if (root->data == data) {
      isDeleted = true;
      // case 1: no children
      if (root->l == nullptr && root->r == nullptr) {
        delete root;
        return nullptr;
      }
        // case 2: has only a child
      else if (root->l == nullptr) {
        delete root;
        return root->r;
      } else if (root->r == nullptr) {
        delete root;
        return root->l;
      }
        // case 3: has two children
      else {
        BstNode<T> *right_min_ptr = _find_min(root->r); // find min in right-tree
        root->data = right_min_ptr->data;
        root->r = _del(root->r, right_min_ptr->data, isDeleted);
        return root;
      }
    }
    if (data < root->data) root->l = _del(root->l, data, isDeleted);
    if (data > root->data) root->r = _del(root->r, data, isDeleted);
  }

  int _find_height(BstNode<T> *root) {
    if (root == nullptr) return -1;
    return std::max(_find_height(root->l), _find_height(root->r)) + 1;
  }

public:
  // TODO - 需要对子树进行翻转处理
  void insert(T data) {
    BstNode<T> *new_node = new BstNode<T>{
            data, nullptr, nullptr
    };
    root_ptr = _insert(root_ptr, new_node);
  }

  bool search(T data) {
    return _search(root_ptr, data);
  }

  T find_max() {
    assert(root_ptr);
    BstNode<T> *tmp = root_ptr;
    while (tmp->r) {
      tmp = tmp->r;
    }
    return tmp->data;
  }

  T find_min() {
    return _find_min(root_ptr)->data;
  }

  int find_height() {
    return _find_height(root_ptr);
  }

  // 遍历树(Binary Tree Traversal)：可以使用广度优先(Breadth-first approach)或深度优先(Depth-first approach)
  // 广度优先：即 Level-order，一排一排遍历。需要借用于 Queue
  // 深度优先搜索包括3种方式：Preorder(DLR)、Inorder(LDR)、Postorder(LRD)
  // 以 Inorder 为例，每个节点都要完成 LDR(left->data->right) 的工作

  void level_order() {
    if (root_ptr == nullptr) return;
    std::queue<BstNode<T> *> q;
    BstNode<T> *tmp = root_ptr;
    q.push(tmp);
    std::cout << "Level order:\t";
    while (tmp != nullptr) {
      std::cout << tmp->data << ' ';
      if (tmp->l) q.push(tmp->l);
      if (tmp->r) q.push(tmp->r);
      q.pop();
      tmp = q.front();
    }
    std::cout << std::endl;
  }

  void preorder() {
    if (root_ptr == nullptr) return;
    std::cout << "Preorder:\t";
    _preorder(root_ptr);
    std::cout << std::endl;
  }

  void inorder() {
    if (!root_ptr) return;
    std::cout << "Inorder:\t";
    _inorder(root_ptr);
    std::cout << std::endl;
  }

  void postorder() {
    if (!root_ptr) return;
    std::cout << "Postorder:\t";
    _postorder(root_ptr);
    std::cout << std::endl;
  }

  bool del(int data) {
    if (root_ptr == nullptr) return false;
    bool isDeleted = 0;
    root_ptr = _del(root_ptr, data, isDeleted);
    return isDeleted;
  }
};


#endif //UNTITLED_BINARYSEARCHTREE_H

```

main:

```c++
#include "BinarySearchTree.h"
#include <iostream>

int fn(int steps) {
  if (steps <= 2) return steps > 0 ? steps : 0;
  return fn(steps - 1) + fn(steps - 2);
}

int main() {
  BinarySearchTree<int> t;
  t.insert(5);
  t.insert(6);
  t.insert(4);
  t.insert(9);
  t.insert(11);
  /*
   * BST:
   *  5
   * 4 6
   *    9
   *     11
   * */
  printf("%d\n", t.search(11));
  printf("%d\n", t.search(4));
  printf("The minimum is %d\n", t.find_min()); // The minimum is 4
  printf("Height is %d\n", t.find_height());
  t.level_order();  // Level order:	5 4 6 9 11
  t.inorder();  // Inorder:	4 5 6 9 11
  t.preorder();  // Preorder:	5 4 6 9 11
  t.postorder();  // Postorder:	4 6 9 11 5
//  printf("Test for isBst: %d\n", isBst<int>(t.root_ptr));
  printf("%d\n", t.del(9));  // 1
  printf("%d\n", t.del(8));  // 0
}

```

## 5. 图

### 介绍

> Graph

图分为有向图和无向图。

有向图：a directed graph or digraph

无向图：a undirected graph

图包括：edges和vertices

写法示例：

G = (V, E)

V = {V1, V2, V3}

E = { {V1, V2}, {V1, V3} }

#### 加权图(weighted graph)

就是edge有权重。对应的是unweighted graph

### 稀疏图(Sparse)和稠密图(Dense)

边很少就是稀疏图，比如边只有顶点那么多。

边很多就是稠密图，比如没有自连接(self-loop)的图，边的数量是最大边 edges * (edges - 1)。

存储稀疏图常用邻接表(Adjacency List)，存储稠密图常用邻接矩阵(Adjacency Matrix)

### 路径(Path)

我们说的路径，顶点是不能重复的。

如果顶点重复的话，称作Walk。比如 `A B D C B E`

### 实现

#### Edge List

> 查找时需要非常耗时。对于稠密图，需要花费O(V^2)

下面是一个加权无向图

```c++
#include <vector>

using std::vector;

struct Edge {
  char start_vertex;
  char end_vertex;
  int weight;
};

int main() {
  vector<char> vertices = {'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'};
  vector<Edge> edges = {
          {'A', 'B', 5},
          {'A', 'C', 7},
          {'A', 'D', 3},
          {'B', 'E', 2},
          {'B', 'F', 10},
          {'C', 'G', 1},
          {'D', 'H', 11},
          {'E', 'H', 9},
          {'F', 'H', 4},
          {'G', 'H', 6},
  };
}
```

#### Adjacency Matrix

> 适合稠密图。时间复杂度很低，为O(V)；但是空间复杂度很高，为O(V^2)

下面是一个非加权无向图

```c++
#include <vector>
#include <map>

using std::vector;
using std::map;

int main() {
2
  vector<vector<int>> edges = {
          {0, 1, 1, 1, 0, 0, 0, 0},
          {1, 0, 0, 0, 1, 1, 0, 0},
          {1, 0, 0, 0, 0, 0, 1, 0},
          {1, 0, 0, 0, 0, 0, 0, 1},
          {0, 1, 0, 0, 0, 0, 0, 1},
          {0, 1, 0, 0, 0, 0, 0, 1},
          {0, 0, 1, 0, 0, 0, 0, 1},
          {0, 0, 0, 1, 1, 1, 1, 0}
  };
}
```

#### Adjacency List

邻接矩阵非常占用空间，而现实生活中大多数都是稀疏图。比如，Facebook有10亿用户，每个人有1K朋友，如果使用邻接矩阵，则每个人需要多存储(1^9 - 1^3)个不需要存储的非朋友。

邻接表就是：存储每个用户的朋友，不存储非朋友。外层是数组，内层可以是链表（就上面学习的数据结构，二叉搜索树比单链表更快）。