# Script Inter-language Bytecode(sbc文件)

字节码sbc文件是将SIR语言转化为二进制文件存储的对应方案。

## SIRBC1.2文件组成

1. 文件头标志
2. 引入定义段(import)
3. 数据定义段(data)
4. 变量定义段(define)
5. 函数定义段(func)
6. 代码段(code)

所有的段信息由4字节长度信息+数据进行存储，即：

| 地址偏移 | 长度 | 数据类型 | 名称 | 描述 |
| ---- | ----- | ---- | ---- | ---- |
| 0x00 | 4 | int | Size | 段长度 |
| 0x04 | Size | byte\[Size\] | Data | 段数据 |


### 1.文件头标志

SIRBC1.2文件的开始8字节用来定义文件头，由"SIRBC"文件标志+三位版本号"1.2"组成。

### 2.引入定义段(import)

引入定义段的起始位置为固定的0x08，读取4字节为引入定义段全部数据的总长度。

```
int importAddr = 8;
int importSize = 0;
importSize = GetInteger(new Span<byte>(sir, importAddr, 4));
```

引入定义的存储方式如下：

| 地址偏移 | 长度 | 数据类型 | 名称 | 描述 |
| ---- | ----- | ----- | ---- | ---- |
| 0x00 | 1 | SirImportTypes | importType | 引入类型 |
| 0x01 | 4 | int | contentLen | 内容长度 |
| 0x05 | contentLen | byte\[contentLen\] | content | 内容 |

所有引入定义依次排列存储，引入定义段的实际存储形式为：

\<段长度\>\[引入定义1\]\[引入定义2\]...\[引入定义n\]

### 3.数据定义段(data)

数据定义段的起始位置为引入定义段起始位置(importAddr)+4+引入定义段数据长度(importSize)，读取4字节为数据定义段全部数据的总长度。

```
int dataAddr = 0;
int dataSize = 0;
dataAddr = importAddr + importSize + 4;
dataSize = GetInteger(new Span<byte>(sir, dataAddr, 4));
```

数据定义的存储方式如下：

| 地址偏移 | 长度 | 数据类型 | 名称 | 描述 |
| ---- | ----- | ---- | ---- | ---- |
| 0x00 | 4 | int | idx | 变量索引号 |
| 0x04 | 1 | SirDataTypes | dataType | 数据类型 |
| 0x05 | 4 | int | dataLen | 数据(UTF8)长度 |
| 0x09 | dataLen | byte\[dataLen\] | data | 数据(UTF8)，数值同样采用字符串方式存储 |

所有数据定义依次排列存储，数据定义段的实际存储形式为：

\<段长度\>\[数据定义1\]\[数据定义2\]...\[数据定义n\]

### 4.变量定义段(define)

变量定义段的起始位置为数据定义段起始位置(dataAddr)+4+数据定义段数据长度(dataSize)，读取4字节为变量定义段全部数据的总长度。

```
int defineAddr = 0;
int defineSize = 0;
defineAddr = dataAddr + dataSize + 4;
defineSize = GetInteger(new Span<byte>(sir, defineAddr, 4));
```

变量定义的存储方式如下：

| 地址偏移 | 长度 | 数据类型 | 名称 | 描述 |
| ---- | ----- | ---- | ---- | ---- |
| 0x00 | 1 | SirScopeTypes | scopeType | 作用域 |
| 0x01 | 4 | int | index | 变量索引号 |
| 0x05 | 4 | int | nameLen | 变量名称(UTF8)长度 |
| 0x09 | nameLen | byte\[nameLen\] | name | 变量名称(UTF8) |

所有变量定义依次排列存储，变量定义段的实际存储形式为：

\<段长度\>\[变量定义1\]\[变量定义2\]...\[变量定义n\]

### 5.函数定义段(func)

函数定义段的起始位置为变量定义段起始位置(defineAddr)+4+变量定义段数据长度(defineSize)，读取4字节为函数定义段全部数据的总长度。

```
int funcAddr = 0;
int funcSize = 0;
funcAddr = defineAddr + defineSize + 4;
funcSize = GetInteger(new Span<byte>(sir, funcAddr, 4));
```

函数定义的存储方式如下：

| 地址偏移 | 长度 | 数据类型 | 名称 | 描述 |
| ---- | ----- | ---- | ---- | ---- |
| 0x00 | 1 | SirScopeTypes | scopeType | 作用域 |
| 0x01 | 4 | int | index | 标签索引号 |
| 0x05 | 4 | int | nameLen | 函数名称(UTF8)长度 |
| 0x09 | nameLen | byte\[nameLen\] | name | 函数名称(UTF8) |

所有函数定义依次排列存储，函数定义段的实际存储形式为：

\<段长度\>\[函数定义1\]\[函数定义2\]...\[函数定义n\]

### 6.代码段(code)

代码段的起始位置为函数定义段起始位置(funcAddr)+4+函数定义段数据长度(funcSize)，读取4字节为代码段全部数据的总长度。

```
int codeAddr = 0;
int codeSize = 0;
codeAddr = funcAddr + funcSize + 4;
codeSize = GetInteger(new Span<byte>(sir, codeAddr, 4));
```

指令的存储方式如下：

| 地址偏移 | 长度 | 数据类型 | 名称 | 描述 |
| ---- | ----- | ---- | ---- | ---- |
| 0x00 | 2 | SirCodeInstructionTypes | ins | 指令类型 |
| 0x02 | 1 | SirExpressionTypes | exp1Type | 参数1类型 |
| 0x03 | 4 | int | exp1Value | 参数1的值 |
| 0x07 | 1 | SirExpressionTypes | exp2Type | 参数2类型 |
| 0x08 | 4 | int | exp2Value | 参数2的值 |
| 0x0C | 1 | SirExpressionTypes | exp3Type | 参数3类型 |
| 0x0D | 4 | int | exp3Value | 参数3的值 |

所有函数定义依次排列存储，函数定义段的实际存储形式为：

\<段长度\>\[指令1\]\[指令2\]...\[指令n\]

