# Reverse--RC4

## 题目描述

小蓝同学发给你一个exe后缀的可执行文件，
你运行后它只输出了一句
话。你意识到这可能不是那么简单，因为更多的信息被隐藏在程序内
部。为了找到隐藏的flag,你需要使用逆向分析技术来深入探究程序的
内部。附件下载提取码(GAME)备用下载

## 解题

可以看到肯定与RC4加密有关

老样子先扔到`exeinfope`中查看：无壳的32位程序

![image-20240220154215533](./img/RC4/image-20240220154215533.png)

放入32位IDA中进行查看，找到`main`函数可以看到程序开始给了我们一个字符串`str=“The quick brown fox jumps over a lazy dog.”`和一个数组`v5`，结尾只有一个`sub_401005`函数，我们对该函数进行跟踪，看到最终执行的加密代码：

​    ![image-20240220154222455](./img/RC4/image-20240220154222455.png)

![image-20240220154228844](./img/RC4/image-20240220154228844.png)

可以看出非常典型的`RC4`加密代码（先进行初始化，再进行加密）可以学习学习`RC4`原理和代码实现，

更据此可以写出最终结题`exp`：（数组`v5`即使待加密的数据，`str`即是密钥`key`）

运行代码得到最终`flag{c8fd99f1-841a-44c9-8d38-746db6ff95c1}`

```c
#include<stdio.h>
void rc4_init(unsigned char* s, unsigned char* key, unsigned long Len_k) //鍒濆鍖栧嚱鏁?
{
    int i = 0, j = 0;
    char k[256] = { 0 };
    unsigned char tmp = 0;
    for (i = 0; i < 256; i++) {
        s[i] = i;
        k[i] = key[i % Len_k];
    }
    for (i = 0; i < 256; i++) {
        j = (j + s[i] + k[i]) % 256;
        tmp = s[i];
        s[i] = s[j];
        s[j] = tmp;
    }
}
void rc4_crypt(unsigned char* Data, unsigned long Len_D, unsigned char* key, unsigned long Len_k) //鍔犺В瀵?
{
    unsigned char s[256];
    rc4_init(s, key, Len_k);
    int i = 0, j = 0, t = 0;
    unsigned long k = 0;
    unsigned char tmp;
    for (k = 0; k < Len_D; k++) {
        i = (i + 1) % 256;
        j = (j + s[i]) % 256;
        tmp = s[i];
        s[i] = s[j];
        s[j] = tmp;
        t = (s[i] + s[j]) % 256;
        Data[k] = Data[k] ^ s[t];
    }
}
int main()
{
    unsigned char key[] = "The quick brown fox jumps over a lazy dog.";
    unsigned long key_len = sizeof(key) - 1;
    unsigned char data[42] = {-120, 118, -41, 122, -76, 90, 76, -57, -36, -17, -55, -114, -38, 89, 14, -104, -113, -5, 11, -44, -30, -79, 19, 96, -26, -57, -122, 43, -36, 113, -44, 48, -19, 42, -119, 97, -125, 0x80, 40, -14, -110, -30};
    rc4_crypt(data, sizeof(data), key, key_len);
    for (int i = 0; i < sizeof(data); i++)
    {
        printf("%c", data[i]);
    }
    printf("\n");
}
```

