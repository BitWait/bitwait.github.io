##  Algorithm:
### 背景：
- 对于一个整数，以uint32为例，其占用4字节，在存储的时候如果都使用4字节来存储uint32类型的数据，有些时候会存在浪费，因为并不是所有整形数据有效位都占用4个字节，例如像0x01用单个字节可以表达，而0x1ff则用2个字节就可以表达。而实际情况中，除了一些hash值等有可能占满4字节，大多数情况下我们的uint32数据是用不到4字节的。为解决这种情况，可以通过一个整形压缩的算法来解决内存/磁盘占用情况。
### 实现方式：
- 使用低7位来表达有效值，高位来表达编码是否完成，如果高位为0代表编码完成，高位为1代表编码未完成，这样在解码的时候便可以通过遍历字节的高位来判断是否完成解码
- 由于有效位只有7位，所以一个uint32的整形最少可以由1个字节完成编码，最多由5个字节完成编码，即 1~5个字节表示；类推uint64位的整形，则可以由 1 ~ 10个字节表示

### 编码实现：
```
void chartobs(char  ch, char* ps)
{
	const int  size = 8 * sizeof(char);
	for (int i = 7; i >= 0; i--, ch >>= 1)
	{
		ps[i] = (01 & ch) + '0';
	}
	ps[size] = '\0';
}

void itobs(int  n, char* ps)
{
	const int  size = 8 * sizeof(int);
	for (int i = size - 1; i >= 0; i--, n >>= 1)
	{
		ps[i] = (01 & n) + '0';
	}
	ps[size] = '\0';
}

void show_bstr(const char* str)
{
	size_t i = 0;
	while (str[i])
	{
		putchar(str[i]);
		if (++i % 8 == 0 && str[i])
			putchar(' ');
	}
	printf("\n");
}
//将一个四字节的整形数值压缩成1~5个字节
size_t encode(unsigned int num, char *buf)
{
	size_t len = 0;
for (int a = sizeof(unsigned int); a >= 0; a--)
	{
		char c;
		c = num >> (a * 7) & 0x7f;
		if (c == 0x00 && len == 0)
		{
			char szDest[16] = { 0 };
			chartobs(c, szDest);
			cout << szDest << " ";
			continue;
		}
		//处理高位标识
		if (a == 0)
			c &= 0x7f;
		else
			c |= 0x80;
		buf[len] = c;
		len++;

		char szDest[16] = { 0 };
		chartobs(c, szDest);
		cout << szDest << " ";
	}
	cout << endl;
	if (len == 0)
	{
		len++;
		buf[0] = 0;
	}
	return len;
}

//将一个1~5个字节的值还原成四字节的整形值
unsigned int decode(char *buf, size_t len)
{
	unsigned int num = 0;
	for (int index = 0; index < (int)len; index++)
	{
		char c = *(buf + index);
		num = num << 7;

		c &= 0x7f;
		num |= c;
	}
	cout << "decode: " << num << endl;
	return num;
}

```
### 测试用例：
```
int main(void)
{
	unsigned int num = 300;
	char szShowStr[64] = { 0 };
	itobs(num, szShowStr);
	show_bstr(szShowStr);

	char szDest[5] = { 0 };
	size_t len = encode(num, szDest);

	system("pause");
	return 0;
}

```
## Review:
### Teach Yourself Programming in Ten Years
- A language that doesn’t affect the way you think about programming, is not worth knowing
- The key is deliberative practice: not just doing it again and again, but challenging yourself with a task that is just beyond your current ability, trying it, analyzing your performance while and after doing it, and correcting any mistakes. Then repeat. And repeat again. There appear to be no real shortcuts:
- Program. The best kind of learning is learning by doing
- Talk with other programmers; read other programs

## tip:
### 指针与引用的区别：引用自：《more effective c++》
- 引用必须被初始化，指针不必
- 引用初始化以后不能被改变，指针可以改变所指的对象
- 不存在指向空值的引用，但是存在指向空值的指针
```
std::string s1("hello");
std::string s2("world");
std::cout <<  "point s1: " << &s1 << " point s2: " << &s2 << std::endl;
std::string& ref = s1;
std::string* point = &s1;
std::cout << "refs: " << ref << " "<<  &ref <<
" points: " << *point << " " << point <<  std::endl;
ref = s2;
point = &s2;
std::cout << "refs: " << ref << " " << &ref <<
" points: " << *point << " " << point << std::endl;
```

## Share:
### 用条件变量实现事件等待器的正确与错误做法 https://blog.csdn.net/Solstice/article/details/11432817

- 本篇文章讲解了条件变量的正确使用方式
### 条件变量的虚假唤醒？
- 因为可能某次操作系统唤醒 pthread_cond_wait 时 tasks.empty() 可能仍然为 true，言下之意就是操作系统可能会在一些情况下唤醒条件变量，即使没有其他线程向条件变量发送信号，等待此条件变量的线程也有可能会醒来。我们将条件变量的这种行为称之为 虚假唤醒 （spurious wakeup）。因此将条件（判断tasks.empty() 为true）放在一个 while 循环中意味着光唤醒条件变量不行，还必须条件满足程序才能继续执行正常的逻辑。
### 为什么会存在虚假唤醒呢？一个原因是
pthread_cond_wait 是 futex 系统调用，属于阻塞型的系统调用，当系统调用被信号中断的时候，会返回 -1，并且把 errno 错误码置为EINTR。很多这种系统调用为了防止被信号中断都会重启系统调用（即再次调用一次这个函数）
除了上面的信号因素外，还存在以下情况：条件满足了发送信号，但等到调用 pthread_cond_wait 的线程得到 CPU 资源时，条件又再次不满足了。好在无论是哪种情况，醒来之后再次测试条件是否满足就可以解决虚假等待的问题。这就是使用 while 循环来判断条件，而不是使用 if 语句的原因。
