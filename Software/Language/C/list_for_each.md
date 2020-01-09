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
