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
