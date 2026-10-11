# C语言文件IO知识整理

## 一、文件IO的两种方式

### 1. 文件IO（系统调用）
- **常见接口**：`open()`、`close()`、`read()`、`write()`、`lseek()`
- **接口来源**：操作系统提供的系统调用接口，通常直接操作文件描述符。
- **缓冲机制**：这些接口本身不提供 C 标准IO的用户态缓冲区；内核仍有自己的缓存机制。
- **特点**：
  - 可以更直接地控制文件读写过程。
  - 每次调用通常会进入内核，调用开销相对明显。
  - 适合需要使用文件描述符、底层控制或特定系统功能的场景。
- **注意**：系统调用不等于“实时性强”，实际实时性还取决于操作系统、设备和调度等因素。

### 2. 标准IO（C标准库）
- **常见接口**：`fopen()`、`fclose()`、`fgetc()`、`fputc()`、`fgets()`、`fputs()`、`fread()`、`fwrite()`、`printf()`等。
- **接口来源**：C标准库封装底层文件操作。
- **缓冲机制**：通常通过`FILE`流提供用户态缓冲区。
- **特点**：
  - 可先在用户态缓冲区中累积数据，再进行底层读写，减少系统调用次数。
  - 提供按字符、按行、按数据块等多种读写方式，使用方便。
  - 需要注意缓冲区刷新与文件关闭。
- **注意**：标准IO最终通常也依赖底层操作系统完成文件操作。

### 3. 两者对比
| 对比项 | 文件IO | 标准IO |
| --- | --- | --- |
| 常见对象 | 文件描述符（整数） | `FILE *`文件流 |
| 常见函数 | `open`、`read`、`write` | `fopen`、`fread`、`fwrite` |
| 用户态缓冲区 | 接口本身不提供标准IO缓冲区 | 通常有 |
| 数据处理方式 | 按字节读写，控制较底层 | 支持字符、行、数据块等 |
| 使用场景 | 底层控制、文件描述符操作 | 一般C语言文件读写 |

## 二、标准IO基础

### 1. `FILE *`文件流指针
- `FILE`是C标准库定义的类型，通常用于表示一个文件流；具体结构由实现决定。
- `FILE *`是指向`FILE`对象的指针。
- `fopen()`成功后返回`FILE *`，后续标准IO函数通过它访问对应文件流。
- 文件流会维护读写状态、缓冲区状态、错误标志、文件结束标志等信息。
- **不要直接依赖或修改`FILE`内部成员**，其内部结构由编译器和C库实现决定。

### 2. 三个预定义标准流
- `stdin`：标准输入，默认通常连接键盘。
- `stdout`：标准输出，默认通常连接终端。
- `stderr`：标准错误输出，默认通常连接终端。
- 它们都是标准IO流，可用于`scanf()`、`printf()`、`fprintf(stderr, ...)`等操作。
- **注意**：标准输入输出也可以被重定向到文件或管道，不一定始终对应键盘和屏幕。

### 3. 常见标准IO函数
- 打开与关闭：`fopen()`、`fclose()`
- 字符读写：`fgetc()`、`fputc()`
- 行读写：`fgets()`、`fputs()`
- 数据块读写：`fread()`、`fwrite()`
- 文件定位：`fseek()`、`ftell()`、`rewind()`
- 状态判断：`feof()`、`ferror()`、`clearerr()`
- 格式化输入输出：`fscanf()`、`fprintf()`、`scanf()`、`printf()`

### 4. 常见文件IO函数
- 打开：`open()`
- 关闭：`close()`
- 读取：`read()`
- 写入：`write()`
- 定位：`lseek()`
- **平台说明**：这些接口主要属于POSIX/Unix类系统接口，不是ISO C标准库函数；Windows环境下通常使用相应平台接口或兼容层。

## 三、`man`手册基础（Linux/Unix）

### 1. 常见章节
- 第1章：可执行程序和Shell命令。
- 第2章：系统调用。
- 第3章：库函数。
- 查看手册：`man man`
- 查看指定章节：`man 3 fopen`、`man 2 open`
- **注意**：章节内容会因系统和发行版有所不同。

## 四、标准IO函数详解

### 1. `fopen()`：打开文件
- **函数原型**：`FILE *fopen(const char *pathname, const char *mode);`
- **头文件**：`#include <stdio.h>`
- **参数**：
  - `pathname`：文件路径和名称，例如`"hello.txt"`、`"../a.c"`、`"/home/linux/xx"`。
  - `mode`：打开模式。
- **常见模式**：
  - `"r"`：只读；文件必须存在；初始位置在文件开头。
  - `"r+"`：读写；文件必须存在；初始位置在文件开头，不会清空文件。
  - `"w"`：只写；文件不存在时创建，存在时清空；初始位置在开头。
  - `"w+"`：读写；文件不存在时创建，存在时清空。
  - `"a"`：追加写；文件不存在时创建；写入内容追加到文件末尾。
  - `"a+"`：读和追加写；文件不存在时创建；读取位置可定位，写入仍追加到末尾。
  - 二进制模式可使用`"rb"`、`"wb"`、`"ab"`等；在区分文本/二进制模式的平台上，两者行为可能不同。
- **返回值**：
  - 成功：返回`FILE *`。
  - 失败：返回`NULL`，通常可结合`perror()`查看系统错误信息。
- **示例**：
  ```c
  #include <stdio.h>

  int main(void)
  {
      FILE *fp = fopen("hello.txt", "r");
      if (fp == NULL) {
          perror("fopen");
          return 1;
      }

      printf("打开成功\n");
      fclose(fp);
      return 0;
  }
  ```
- **易错点**：
  - `"r"`和`"r+"`不会自动创建文件。
  - `"w"`和`"w+"`会清空已存在文件，使用前要确认。
  - 打开成功后应在适当位置调用`fclose()`。

### 2. `fclose()`：关闭文件
- **函数原型**：`int fclose(FILE *stream);`
- **头文件**：`#include <stdio.h>`
- **功能**：关闭文件流；对于输出流，会尝试刷新尚未写出的缓冲数据。
- **参数**：`stream`为此前成功打开且尚未关闭的文件流。
- **返回值**：
  - 成功：返回`0`。
  - 失败：返回`EOF`。
- **示例**：
  ```c
  if (fclose(fp) == EOF) {
      perror("fclose");
      return 1;
  }
  fp = NULL;
  ```
- **易错点**：
  - 不要重复关闭同一个文件流。
  - 关闭后不能继续使用该`FILE *`。
  - 写文件时，错误可能在`fclose()`刷新缓冲区时才被报告。

### 3. `perror()`：打印错误原因
- **函数原型**：`void perror(const char *s);`
- **头文件**：`#include <stdio.h>`
- **功能**：将提示字符串和当前`errno`对应的错误信息输出到标准错误流`stderr`。
- **示例**：
  ```c
  FILE *fp = fopen("missing.txt", "r");
  if (fp == NULL) {
      perror("fopen");
      return 1;
  }
  ```
- **注意**：
  - `perror()`没有返回值。
  - 应在发生错误后尽快调用，避免后续操作改变错误状态。
  - 它适合报告系统/库函数错误，不适合用来检查错误；检查返回值才是关键。

### 4. `fgetc()`：读取一个字符
- **函数原型**：`int fgetc(FILE *stream);`
- **头文件**：`#include <stdio.h>`
- **功能**：从文件流读取下一个字符。
- **返回值**：
  - 成功：返回读取字符转换成的`unsigned char`值，再转换为`int`。
  - 到达文件末尾或发生错误：返回`EOF`。
- **为什么返回`int`**：需要既能表示所有字符值，也能表示特殊值`EOF`。
- **示例**：
  ```c
  int ch;
  while ((ch = fgetc(fp)) != EOF) {
      putchar(ch);
  }

  if (ferror(fp)) {
      perror("fgetc");
  }
  ```
- **易错点**：
  - 用`int`接收返回值，不要用`char`判断`EOF`。
  - 不要用`while (feof(fp))`作为读取循环条件；文件结束标志通常是在一次读取失败后才设置。
  - 推荐“先读取，再判断返回值”的写法。

### 5. `fputc()`：写入一个字符
- **函数原型**：`int fputc(int c, FILE *stream);`
- **头文件**：`#include <stdio.h>`
- **功能**：将字符`c`写入文件流。
- **参数**：
  - `c`：待写入的字符值。
  - `stream`：可写的文件流。
- **返回值**：
  - 成功：返回写入字符转换后的`unsigned char`值。
  - 失败：返回`EOF`。
- **示例**：
  ```c
  if (fputc('a', fp) == EOF) {
      perror("fputc");
  }
  ```
- **易错点**：
  - 文件必须以可写模式打开。
  - 注意检查返回值，并在结束时检查`fclose()`是否失败。

### 6. `feof()`与`ferror()`：检查流状态
- **函数原型**：
  ```c
  int feof(FILE *stream);
  int ferror(FILE *stream);
  ```
- **`feof()`**：
  - 返回非0：文件结束标志已设置。
  - 返回0：文件结束标志未设置。
  - 通常在读取操作因到达末尾而失败后，才会设置文件结束标志。
- **`ferror()`**：
  - 返回非0：流的错误标志已设置。
  - 返回0：未检测到流错误。
- **正确读取模式**：
  ```c
  int ch;
  while ((ch = fgetc(fp)) != EOF) {
      printf("%c", ch);
  }

  if (ferror(fp)) {
      perror("读取文件");
  } else if (feof(fp)) {
      printf("读取结束\n");
  }
  ```
- **易错点**：
  - `fgetc()`返回`EOF`既可能是文件结束，也可能是读取错误，要进一步判断。
  - `feof()`不是预测“下一次读取是否会结束”的函数。
  - 若要清除流的结束/错误标志，可使用`clearerr()`。

## 五、命令行参数

### 1. `argc`与`argv`
- 常见主函数形式：`int main(int argc, char *argv[])`
- `argc`：命令行参数数量，通常包含程序名。
- `argv`：字符串指针数组，保存每个命令行参数。
- `argv[0]`：通常是程序名或调用路径。
- `argv[1]`及后续元素：用户传入的参数。
- **访问前检查参数数量**，避免访问不存在的参数。
- **示例**：
  ```c
  #include <stdio.h>

  int main(int argc, char *argv[])
  {
      printf("argc = %d\n", argc);

      for (int i = 0; i < argc; i++) {
          printf("argv[%d] = %s\n", i, argv[i]);
      }

      if (argc < 2) {
          fprintf(stderr, "用法：%s 文件名\n", argv[0]);
          return 1;
      }

      FILE *fp = fopen(argv[1], "r");
      if (fp == NULL) {
          perror("fopen");
          return 1;
      }

      fclose(fp);
      return 0;
  }
  ```
- **易错点**：直接使用`argv[1]`前，应先判断`argc >= 2`。

## 六、文件IO编程常见错误与记忆要点

### 1. 常见错误
- `fopen()`失败后没有判断`NULL`，继续使用文件指针。
- 使用`"w"`打开文件，却忘记它会清空原内容。
- 把`feof()`放在读取循环条件中，导致多读一次或输出异常。
- 用`char`保存`fgetc()`返回值，导致无法可靠区分`EOF`。
- 只检查读写函数，不检查`fclose()`可能出现的刷新错误。
- 文件操作结束后忘记关闭文件。
- 发生错误时只打印固定字符串，不使用`perror()`等方式辅助定位原因。

### 2. 推荐的文件读取模板
```c
#include <stdio.h>

int main(void)
{
    FILE *fp = fopen("hello.txt", "r");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }

    int ch;
    while ((ch = fgetc(fp)) != EOF) {
        putchar(ch);
    }

    if (ferror(fp)) {
        perror("读取文件");
        fclose(fp);
        return 1;
    }

    if (fclose(fp) == EOF) {
        perror("fclose");
        return 1;
    }

    return 0;
}
```

### 3. 速记
- **打开**：`fopen()`，先判断是否为`NULL`。
- **读取字符**：`fgetc()`，用`int`接收，循环判断是否为`EOF`。
- **写入字符**：`fputc()`，检查是否返回`EOF`。
- **判断结束/错误**：`feof()`判断结束标志，`ferror()`判断错误标志。
- **报告错误**：`perror()`输出当前错误原因。
- **关闭文件**：`fclose()`，检查是否返回`EOF`。
- **文件IO**使用文件描述符；**标准IO**使用`FILE *`文件流。
