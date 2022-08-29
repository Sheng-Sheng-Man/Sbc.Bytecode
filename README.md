# Script Inter-language Bytecode(sbc文件)

字节码sbc文件是将SIR语言转化为二进制文件存储的对应方案。

## sbc文件组成

1. 文件头标志
2. 引入定义段(import)
3. 数据定义段(data)
4. 变量定义段(define)
5. 函数定义段(func)
6. 代码段(code)

所有的段信息由4字节长度信息+数据进行存储，即：

| 地址偏移 | 长度 | 名称 | 描述 |
| ---- | ----- | ---- | ---- |
| 0x00 | 4 | Size | 段长度 |
| 0x04 | Length | Data | 段数据 |


### 文件头标志

sbc文件的开始8字节用来定义文件头，由"SIRBC"+三位版本号组成。

### 引入定义段(import)

引入定义段的起始位置为固定的0x08，读取4字节为引入定义段全部数据的总长度。

```
int importAddr = 8;
int importSize = 0;
importSize = GetInteger(new Span<byte>(sir, importAddr, 4));
```

引入数据的存储方式如下：

| 地址偏移 | 长度 | 名称 | 描述 |
| ---- | ----- | ---- | ---- |
| 0x00 | 1 | importType | 引入类型 |
| 0x01 | 4 | contentLen | 内容长度 |
| 0x05 | contentLen | content | 内容 |

所有引入数据依次排列存储，引入定义段的实际存储形式为：

\<段长度\>\[引入数据1\]\[引入数据2\]...\[引入数据n\]

