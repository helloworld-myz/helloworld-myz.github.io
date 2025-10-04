+++
date = '2025-10-03T09:36:06+08:00'
draft = false
title = 'Python 实现管理员权限运行'
+++

# Python 使用管理员权限运行，不需要手动操作

在写代码时难免会涉及到系统文件的读写、执行，这时候就不可避免地涉及到了管理员权限。
如果让用户手动右键选择使用管理员权限运行，又太麻烦。有没有更好的解决方案呢？

### 注意：没有Python环境的请提前安装Python环境
### 因为代码中使用的ctypes.windll.shell32(Windows系统接口)，所以此方法仅适用于Windows系统。

### 需要的代码：
```python
import ctypes
import sys
import subprocess as sp

def run_as_admin(target_path):
    """以管理员权限运行指定的目标文件（支持Python脚本或可执行文件）"""
    if ctypes.windll.shell32.IsUserAnAdmin():
        # 已获得管理员权限，直接执行目标
        try:
            sp.run(target_path, check=True)
            print(f"成功执行：{target_path}")
        except Exception as e:
            print(f"执行失败：{e}")
    else:
        # 未获得管理员权限，请求UAC提权
        # 处理路径包含空格的情况（用引号包裹）
        quoted_path = f'"{target_path}"'
        
        # 根据文件类型构造命令（Python脚本需要指定解释器）
        if target_path.endswith(".py"):
            cmd = f'"{sys.executable}" {quoted_path}'  # 调用Python解释器执行脚本
        else:
            cmd = quoted_path  # 直接运行可执行文件
        
        # 提权执行命令
        result = ctypes.windll.shell32.ShellExecuteW(
            None, "runas", "cmd.exe", f"/c {cmd}", None, 1
        )
        if result <= 32:
            print("请求管理员权限失败，请手动以管理员身份运行")

if __name__ == "__main__":
    # 仅处理传入1个参数的情况（目标文件路径）
    if len(sys.argv) == 2:
        target = sys.argv[1]
        run_as_admin(target)
    else:
        # 参数数量错误时提示用法
        print(f"用法：{sys.argv[0]} <目标文件路径>")
```
### 使用Pyinstaller打包(w参数可选)：
```bash
pyinstaller -F 代码文件.py
```
完成之后，打开dist目录，里面的exe文件就是我们需要的。

### 如何使用
- 先通过 dir 查看 dist 目录下生成的 .exe 文件名（如 run_as_admin.exe）。
- 在命令行中输入 .exe 文件名，加空格，再输入目标文件路径（路径含空格必须用引号包裹）。
### 小技巧
- 找到需要运行的程序，右键→“属性”→“目标”，即可复制完整路径，然后用引号包裹使用。
- 更简单：直接把目标文件拖到生成的 run_as_admin.exe 上，会自动以管理员权限运行（无需手动输入路径）。
```bash
cd dist
run_as_admin.exe cmd.exe
run_as_admin.exe "C:\Program Files\Windows NT\Accessories\wordpad.exe"
```

### 这有效果吗？
首先，请出我们接近操作系统的C++来检测程序是否起效！
```c++
#include <ShlObj.h>
#include <iostream>

using namespace std;
int main()
{
    bool bIsAdmin = IsUserAnAdmin();
    if (bIsAdmin)
        cout << "拥有管理员权限" << endl;
    else
        cout << "没有管理员权限" << endl;
    system("pause");
    return 0;
}
```
效果图：
![拥有管理员权限时](/images/Python使用管理员身份运行/有管理员.png)
![没有管理员权限时](/images/Python使用管理员身份运行/无管理员.png)
### 正式开始测试
![开始测试](/images/Python使用管理员身份运行/运行.png)
结果出来了，C++程序获得了管理员权限！
![获得了管理员权限](/images/Python使用管理员身份运行/结果.png)
### 总结
这篇文章连我自己都觉得充满水分😅