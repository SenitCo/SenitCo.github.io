---
title: C/C++处理十六进制数和字符串
date: 2017-06-07
categories: coding
tags: C/C++ 
---

&emsp;&emsp;C/C++处理十六进制数和字符串小结，包括十六进制数组和字符串的相互转换，二进制字符串和十六进制数组的转换，不定长字符串的读取等。
<!-- more -->

### 十六进制数组和字符串的相互转换
例如 { 0x23, 0x3A, 0x46, 0x4C, 0x52 } <=> "233A464C52"
```
/******************************************************************************************
*	功能：将一个十六进制字节串转换成 ASCII 码表示的十六进制的字符串
*	输入参数：pHex	 -- 十六进制数字节串首地址
*			 pAscii -- 转换后的 ASCII 码表示的十六进制字符串的首地址
*			 nLen	 -- 要转换的十六进制数的长度（字节数）
*	输出参数：None
*	注：	转换后的结果全部是大写 ASCII 表示的十六进制数
*******************************************************************************************/
void HexToAscii(unsigned char * pHex, unsigned char * pAscii, int nLen)
{
    unsigned char Nibble[2];

    for (int i = 0; i < nLen; i++)
    {
        Nibble[0] = (pHex[i] & 0xF0) >> 4;
        Nibble[1] = pHex[i] & 0x0F;
        for (int j = 0; j < 2; j++)
        {
            if (Nibble[j] < 10)
                Nibble[j] += 0x30;
            else
            {
                if (Nibble[j] < 16)
                    Nibble[j] = Nibble[j] - 10 + 'A';
            }
            *pAscii++ = Nibble[j];
        }	// for (int j = ...)
    }	// for (int i = ...)
}

/******************************************************************************************
*	功能：将一个 ASCII 码表示的十六进制字符串转换成十六进制的字节串
*	输入参数：pAscii -- 转换后的 ASCII 码表示的十六进制字符串的首地址
*			 pHex	-- 十六进制数字节串首地址
*			 nLen	-- 要转换的 ASCII 码表示的十六进制字符串的长度（字节数）
*	输出参数：None
*	注：	要求输入的 ASCII 码表示的十六进制数的字符个数必须为偶数，除了是1 - 9 和 A(a) - F(f) 以外没有别的字符
*******************************************************************************************/
void AsciiToHex(unsigned char * pAscii, unsigned char * pHex, int nLen)
{
    if (nLen % 2)
        return;
    int nHexLen = nLen / 2;

    for (int i = 0; i < nHexLen; i++)
    {
        unsigned char Nibble[2];
        Nibble[0] = *pAscii++;
        Nibble[1] = *pAscii++;
        for (int j = 0; j < 2; j++)
        {
            if (Nibble[j] <= 'F' && Nibble[j] >= 'A')
                Nibble[j] = Nibble[j] - 'A' + 10;
            else if (Nibble[j] <= 'f' && Nibble[j] >= 'a')
                Nibble[j] = Nibble[j] - 'a' + 10;
            else if (Nibble[j] >= '0' && Nibble[j] <= '9')
                Nibble[j] = Nibble[j] - '0';
            else
                return;
        }	// for (int j = ...)
        pHex[i] = Nibble[0] << 4;	// Set the high nibble
        pHex[i] |= Nibble[1];	//Set the low nibble
    }	// for (int i = ...)
}
```

### 十六进制数组和二进制字符串的相互转换
例如 { 0x23, 0x4A, 0x5E } <=> "001000110100101001011110"
```
/***十六进制数转换成二进制字符串***/
void HexToBinStr(unsigned char* hexStr, unsigned char* binStr, int lenHex)
{
    memset(binStr, '0', lenHex * 8);
    unsigned char hexChar[2];
    for (int i = 0; i < lenHex; i++)
    {
        hexChar[0] = (hexStr[i] & 0xF0) >> 4;
        hexChar[1] = hexStr[i] & 0x0F;
        for (int j = 0; j < 2; j++)
        {
            for (int k = 0; k < 4; k++)
            {
                if (hexChar[j] & (0x08 >> k))
                {
                    binStr[8 * i + 4 * j + k] = '1';
                }
            }
        }
    }
}

/***二进制字符串转换成十六进制数***/
void BinStrToHex(unsigned char* binStr, unsigned char* hexStr, int lenBin)
{
    int lenHex = lenBin / 8;
    memset(hexStr, '\0', lenHex);
    unsigned char hexChar[2];
    for (int i = 0; i < lenHex; i++)
    {
        for (int j = 0; j < 2; j++)
        {
            hexChar[j] = 0;
            for (int k = 0; k < 4; k++)
            {
                if (binStr[8 * i + 4 * j + k] == '1')
                {
                    hexChar[j] |= (0x08 >> k);
                }
            }
        }
        hexStr[i] = ((hexChar[0] & 0x0F) << 4) | (hexChar[1] & 0x0F);
    }
}
```

### 读取不定长字符串
```
/*******************读取不定长字符串******************/
unsigned char* getlineStr()
{
	int nByte = 50;
    char * line = (char *)malloc(nByte), *linep = line;
    size_t lenmax = nByte, len = lenmax;
    int c;

    if (line == NULL)
        return NULL;

    for (;;)
    {
        c = fgetc(stdin);
        if (c == EOF)
            break;

        if (--len == 0)
        {
            len = lenmax;
            char * linen = (char *)realloc(linep, lenmax *= 2);

            if (linen == NULL)
            {
                free(linep);
                return NULL;
            }
            line = linen + (line - linep);
            linep = linen;
        }

        if ((*line++ = c) == '\n')
            break;
    }
    *--line = '\0';     //用'\0'替换掉换行'\n'
    return (unsigned char*)linep;
}
```

### 奇偶校验
```
/***奇偶校验，使每个字节比特1的个数为奇数个***/
void checkParity(unsigned char* srcChar, unsigned char* dstChar, int nLen)
{
    unsigned char sinChar;
    short minBit = 0;
    short count = 0;
    for (int i = 0; i < nLen; i++)
    {
        count = 0;
        sinChar = srcChar[i];
        minBit = sinChar % 2;
        for (int j = 0; j < 8; j++)
        {
            if (sinChar % 2 == 1)
                count++;
            sinChar >>= 1;
        }
        if (count % 2 == 1)
            dstChar[i] = srcChar[i];
        else if (minBit == 1)
            dstChar[i] = srcChar[i] - 1;
        else
            dstChar[i] = srcChar[i] + 1;
    }
}
```

### 字节填充(nByte字节的整数倍)
```
/***********************************************************************************
* 字节填充（8字节的整数倍）
* 输入：binStr 二进制字符串; nByte 字节数
* 返回：fillStr 填充后的字符串
* 填充方法：在最右端填充一个‘1’位，之后再填充若干‘0’，直到该数据的最终字节数为 nByte 的整数倍
************************************************************************************/
unsigned char* fillByte(unsigned char* binStr, int nByte)
{
    int nBit = nByte * 8;
    int len1 = 0, len2 = 0;
    unsigned char* fillStr;
    unsigned char* tmpStr;
    if (strlen((const char*)binStr) % nBit == 0)
    {
        len1 = nBit;
    }
    else
    {
        len1 = nBit - strlen((const char*)binStr) % nBit;
    }
    len2 = strlen((const char*)binStr) + len1;
    tmpStr = (unsigned char*)malloc((len1 + 1) * sizeof(unsigned char));
    fillStr = (unsigned char*)malloc((len2 + 1) * sizeof(unsigned char));
    if (fillStr == NULL)
    {
        printf("allocation failture\n");
        exit(0);
    }
    memset(tmpStr, '0', len1);
    tmpStr[0] = '1';
    tmpStr[len1] = '\0';

    memcpy(fillStr, binStr, strlen((const char*)binStr));
    memcpy(fillStr + strlen((const char*)binStr), tmpStr, len1);
    fillStr[len2] = '\0';

    return fillStr;
}
```

### 以十六进制数形式读取文件
文件是以字符形式读取的，因此需要转换为对应的十六进制数(例如"5C"->0x5C)
```
/*****************单个字符转为数字('A'->10)*******************/
static unsigned int hex_char_to_dec(char c)
{
    if ('0' <= c && c <= '9')
    {
        return (c - '0');
    }
    else if ('a' <= c && c <= 'f')
    {
        return (c - 'a' + 10);
    }
    else if ('A' <= c && c <= 'F')
    {
        return (c - 'A' + 10);
    }
    else
    {
        return -1;
    }
}

/************两个字符转为一个16进制数("4B"->0x4B)**************/
static unsigned int str_to_hex(const char *str)
{
    return (str[1] == '\0') ? hex_char_to_dec(str[0]) : hex_char_to_dec(str[0]) * 16 + hex_char_to_dec(str[1]);
}

/*
*对字符串inputString按tag字符分割
*返回vector<string>格式的一维向量
*/
vector<string> split(string inputString, char tag)
{
    int length = inputString.length();
    int start = 0;//数值起始下标
    vector<string> line;
    for (int i = 0; i<length; i++)
    {
        if (inputString[i] == tag)
        {//遇到tag字符
            string sub = inputString.substr(start, i - start);    //取inputString[start]-inputString[i]子串
            line.push_back(sub);//压入向量中
            start = i + 1;
        }
        else if (i == length - 1)
        {
            string sub = inputString.substr(start, i - start + 1);//最后一个字符没有标点，需单独处理
            line.push_back(sub);//压入向量中
        }
    }
    return line;
}

/*
*读取绝对路径为filePath的文件，文件中每行中的数值以tag字符分开
*返回字节数
*/
int readFile(char tag, string filePath, unsigned char* data)
{
    ifstream fileReader;
    fileReader.open(filePath, ios::in);//以只读方式打开
    vector<vector<string>> vecData;//以2维向量的形势保持整个文件
    int i = 0;
    while (!fileReader.eof())
    {//未到文件末尾    
        string linestring;
        getline(fileReader, linestring);//读取一行
        vector<string> line = split(linestring, tag);//分割每行,并放在line向量中    
        for (vector<string>::iterator iter = line.begin(); iter != line.end(); iter++)
        {
            data[i] = str_to_hex(iter->c_str());
            i++;
        }
        vecData.push_back(line);
    }
    //return vecData;
    return i + 1;
}
```
