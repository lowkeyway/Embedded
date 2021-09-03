# 1、strcat()
此函数原型为 char *strcat(char *dest, const char *src).
功能为连接两个字符串，把src连接到dest后面；返回dest地址
实现如下
```
char * strcat(char *dest,const char *src)
{
	char* addr=dest;
	while(*dest)//找到'\0'
	{
		dest++;
	};
	while(*dest++=*src++)
	{};
	return addr;
}
```

# 2、strcmp()
此函数的函数原型为 int strcmp(const char *str1, const char *str2).
功能为比较两个字符串。
当str1指向的字符串大于str2指向的字符串时，返回正数。
当str1指向的字符串等于str2指向的字符串时，返回0。
当str1指向的字符串小于str2指向的字符串时，返回负数。
实现如下：
```
int strcmp(const char *str1, const char *str2)
{
	while(*str1==*str2)
	{
		if(*str1=='\0')
			return 0;
		str1++;
		str2++;
	}
	return *str1-*str2;
}
```

# 3、strcpy()
此函数原型为 char *strcpy(char* dest, const char *src)
功能为拷贝字符串内容到目的串，把src所指向的内容拷贝到dest
实现如下
```
char *strcpy(char *dest,const char *src)
{
	//assert(dest!=NULL&&src!=NULL);
	char *addr=dest;
	while(*dest++=*src++);
	return addr;
}
```
 
# 4、strlen()  
此函数原型为unsigned in strlen(const char *str)

功能为返回字符串str的长度（不包括'\0')。

实现如下：
```
unsigned int strlen(const char *str)
{
	unsigned len=0;
	while(*str!='\0')
	{
		len++;
		str++;
	}
	return len;
}
``` 

# 5、strchr()  strrchr()

## char *strchr(char *str, char c)

功能为查找str中首次出现c的位置，如有有，则返回出现位置，否则返回NULL。实现如下：
```
char *strchr(char *str, char c)
{
	while(*str!='\0'&&*str!=c)
	{
		str++;
	}
	return (*str==c? str: NULL);
}
```

## char *strrchr(char *str, char c)

功能为查找str中最后一次出现c的位置，如有有，则返回出现位置，否则返回NULL。实现如下：
```
char *strrchr(char *str, char c)
{
	char *p=str+strlen(str);//p指向最后一个字符
	while(p!=str&&*p!=c)
	p--;
	if(p==str&&*p!=c)
		return NULL;
	else return p;
}
```

# 6、strcspn()  strspn()

## strcspn

原型：size_t strcspn(const char *pstr, const char *strCharset)

MSDN解释为：在字符串pstr中搜寻strCharsret中所出现的字符，返回strCharset中出现的第一个字符在pstr中的出现位置。简单的说，若strcspn返回的数值为n，则代表字符串strCharsrt开头连续有n个字符不包含在pstr内的字符。

实现十分巧妙，在http://blog.csdn.net/chenyu2202863/article/details/5293941

原型size_t strspn(const char *pstr, const char *strCharset)

功能：返回后面字符串中第一个不在前者出现的下表。 

# 7、strdup()
此函数原型为char *strdup(const char *str)
功能为拷贝字符串到新建的内存，返回内存指针。若失败，返回NULL。要注意，返回的指针指向的内存在堆中，所以要手动释放。
函数实现：
```
char *strdup(const char *str)
{
	char *p=NULL;
	if(str&&(p=(char*)malloc(strlen(str)+1)))
		strcpy(p,str);
	return p;
}
```

# 8、strrev()
此函数的原型为char *strrev(char *str)
功能为反转字符串，返回字符串指针。
函数实现：
```
char *strrev(char *str)
{
	if(str==NULL)
		return NULL;
	char *start=str;
	char *end=str+strlen(str)-1;
	char temp;
	while(start<end)
	{
		temp=*start;
		*start=*end;
		*end=temp;
		start++;
		end--;
	}
	return str;
}
```

# 9、strstr()
函数原型为char *strstr(const char str1, const char *str2)
功能为查找字符串str2在str1中出现的位置，找到则返回位置，否则返回NULL 。
函数实现：
```
char *strstr(const char str1, const char *str2)
{
	int length1=strlen(str1);
	int length2=strlen(str2);
	while(length1>=length2)
	{
		length1--;
		if(!strncpy(str1,str2,length2))//比较前n个字符串，类似strcpy
			return str1;
		str1++;
	}
	return NULL;
}
```
