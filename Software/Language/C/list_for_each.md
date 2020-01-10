# list_head

内核中经常采用链表来管理对象，先看一下内核中对链表的定义
    struct list_head {
        struct list_head *next, *prev;
    };

一般将该数据结构嵌入到其他的数据结构中，从而使得内核可以通过链表的方式管理新的数据结构，看一个例子:
```
struct example {
    member a;
    struct list_head list;
    member b;
};
```    


#  list_entry

前面说过，list_head结构通常被嵌入到其他数据结构中，以便内核可以通过链表的方式管理这些数据结构。假设这样一种场景：我们已知类型为example的对象的list成员的地址ptr（struct list_head *ptr），那么我们如何通过ptr来得到该example对象的地址呢？答案很明显，使用container_of宏。不过，在这样的情况下我们应该通过使用list_entry宏来完成container_of宏的功能，因为这样更容易理解一点。其实list_entry宏很简单：#define list_entry(ptr, type, member)  container_of(ptr, type, member) ......

上述情况，我们可以这样： list_entry(ptr, struct example, list); 来获取example对象的指针。

# list_for_each

Linux 中的源代码。

```
/**
 * list_for_each	-	iterate over a list
 * @pos:	the &struct list_head to use as a loop cursor.
 * @head:	the head for your list.
 */
#define list_for_each(pos, head)  for (pos = (head)->next; pos != (head); pos = pos->next)
```


# list_for_each_entry

双向链表及链表头:

建立一个双向链表通常有一个独立的用于管理链表的链表头，链表头一般是不含有实体数据的，必须用INIT_LIST_HEAD（）进行初始化，表头建立以后，就可以将带有数据结构的实体链表成员加入到链。

```
INIT_LIST_HEAD (&nphy_dev_list);
```

定义:
```
#define list_for_each_entry(pos, head, member)                \
    for (pos = list_entry((head)->next, typeof(*pos), member);    \
         &pos->member != (head);     \
         pos = list_entry(pos->member.next, typeof(*pos), member))
```
它实际上是一个 for 循环，利用传入的 pos 作为循环变量，从表头 head 开始，逐项向后（next 方向）移动 pos，直至又回head.


我们将for循环分解为一下三点：

1. for循环初始化      pos = list_entry((head)->next, typeof(*pos), member);
2. for循环执行条件  &pos->member != (head);
3. 每循环一次执行   pos = list_entry(pos->member.next, typeof(*pos), member))


typeof()是取变量的类型，这里是取指针pos所指向数据的类型。

先看第一个：
```
#define list_entry(ptr, type, member) \
    container_of(ptr, type, member)
```
```
#define container_of(ptr, type, member) ({                      \
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    (type *)( (char *)__mptr - offsetof(type,member) );})
```


讲讲container_of：
作用：根据一个结构体变量中的一个域成员变量的指针来获取指向整个结构体变量的指针。

所以(type \*)0)就是将0强转为一个地址，这个地址(0x0000)指向的是类型type的数据。当然，这里是一个技巧，并不是真的在地址0x0000存放了我们的数据。

((type \*)0)->member的作用，这里的‘->’很显然是通过指针指取结构体成员的操作。指针就是刚才通过0强转的地址。所以也就是相当于地址0x0000 是结构体类型type的首地址，通过->’取其中的成员变量member。

typeof( ((type \*)0)->member ) \*__mptr = (ptr)：知道member成员的类型，定义一个指针变量__mptr，指向的类型是member的类型，其初始化为ptr的值

offsetof(type,member):求出member在结构体中的偏移量

```
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

根据优先级的顺序，最里面的小括号优先级最高，TYPE *将整型常量0强制转换为TYPE型的指针，且这个指针指向的地址为0，也就是将地址0开始的一块存储空间映射为TYPE型的对象，接下来再对结构体中MEMBER成员进行取址，而整个TYPE结构体的首地址是0，这里获得的地址就是MEMBER成员在TYPE中的相对偏移量。再将这个偏移量强制转换成size_t型数据（无符号整型）。  
