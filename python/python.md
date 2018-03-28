# python 知识点

## python 让 json dumps 输出中文

  `json.dumps({'text':"中文"},ensure_ascii=False)`

## python 搜索包路径
    - 1、程序的根目录
    - 2、PYTHONPATH环境变量设置的路径
    - 3、标准库的路径
    - 4、任何能找到的.pth文件的内容
    - 5、第三方的site-package目录

这五部分拼接组成了`sys.path`， 其中第一，第三和第五是自动定义的。

**根目录（自动）**

当运行一个程序时，这个入口就是程序所在的目录，

**PYTHONPATH目录（可配置）**

搜索PYTHONPATH环境变量所在目录的包

**标准库（自动）**
**.pth文件列出的目录(可配置)**

在 /usr/local/lib/python3.5/site-packages 下添加一个扩展名为 .pth 的配置文件（例如：extras.pth），内容为要添加的路径：（每行一个）`/home/wang/workspace`

**lib/site-package目录(自动)**

python会在搜索路径上自动加上site-packages目录，这一般是第三方扩展安装的地方，一般是由distutils工具发布的。


**向python添加一个路径**

```python
import sys
sys.path.append("/home/test/test/")
```







