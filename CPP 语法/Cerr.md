`cerr` 是 C++ 里的**标准错误输出流**，全名可以理解成：

```cpp
std::cerr
```

它通常用来输出**错误信息、警告信息、调试提示**。

例如：

```cpp
#include <iostream>
using namespace std;

int main()
{
    cerr << "参数数量错误！" << endl;
    return 1;
}
```

这里：

```cpp
cerr << "参数数量错误！" << endl;
```

意思是：

```cpp
把错误信息输出到标准错误流。
```

---

它和 `cout` 很像：

```cpp
cout << "hello" << endl;
cerr << "error" << endl;
```

区别是：

|名称|用途|
|---|---|
|`cout`|正常输出|
|`cerr`|错误输出|
|`cin`|标准输入|

比如程序正常结果应该用：

```cpp
cout << result << endl;
```

错误提示应该用：

```cpp
cerr << "Usage: " << argv[0] << " HOST PATH\n";
```

---

为什么不用 `cout` 输出错误？

因为操作系统把**正常输出**和**错误输出**分成了两个通道：

```cpp
cout -> stdout 标准输出
cerr -> stderr 标准错误
```

这样可以分开重定向。

例如：

```bash
./program > output.txt
```

这只会把 `cout` 的内容写进 `output.txt`，但 `cerr` 的错误信息仍然会显示在终端。

如果想把错误也重定向：

```bash
./program > output.txt 2> error.txt
```

含义是：

```bash
正常输出 -> output.txt
错误输出 -> error.txt
```

---

所以你在 CS144 代码里看到：

```cpp
cerr << "Usage: " << argv[0] << " HOST PATH\n";
return EXIT_FAILURE;
```

它的意思是：

```cpp
参数用错了，向标准错误流打印使用说明，然后返回失败退出码。
```

一句话记：

> `cerr` 就是专门输出错误信息的 `cout`。