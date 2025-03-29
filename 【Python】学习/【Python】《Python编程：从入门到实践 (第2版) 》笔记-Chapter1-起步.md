###### 一、起步

1. 本书编写期间的最新版本为Python 3.7，但只要你安装了Python 3.6或更高的版本，就能运行本书中的所有代码。

2. 运行Python代码片段
- Python自带一个在终端窗口中运行的解释器，让你无须保存并运行整个程序就能尝试运行Python代码片段。
    ```
    C:\> python --version
    Python 3.7.0
    C:\> python
    Python 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)] on win32
    Type "help", "copyright", "credits" or "license" for more information.  
    >>> 
    >>> print("Hello Python interpreter!") 
    Hello Python interpreter!
    >>> 
    ```
    - 提示符 >>> 表明正在使用终端窗口。

3. 在Windows系统中搭建Python编程环境
- 请务必选中复选框Add Python（版本号）to PATH，这让你能够更轻松地配置系统。
- 每当要运行Python代码片段时，都请打开一个命令窗口并启动Python终端会话。要关闭该终端会话，可按Ctrl + Z、再按回车键，也可执行命令exit() 。

4. 文件名和文件夹名称最好使用小写字母，并使用下划线代替空格，因为Python采用了这些命名约定。

5. Windows 系统使用命令dir（表示directory，即目录）可以显示当前目录中的所有文件。

6. 从终端运行Python程序
```
C:\> cd Desktop\python_work
C:\Desktop\python_work> dir 
hello_world.py
C:\Desktop\python_work> python hello_world.py 
Hello Python world!
```
- 要运行Python程序，只需使用命令python（或python3）即可。
