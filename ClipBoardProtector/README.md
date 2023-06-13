# 加密视频播放软件会恶意占用剪贴板不让使用。导致一些正常的跟着视频操作的行为都没法完成。
# 解决方法：
```
pip install pyperclip
```
之后，在加密视频软件运行起来之前，先占用上剪贴板资源（不然在之后运行，python程序就会报错“拒绝访问”。）
运行下面python代码，然后在打开加密视频播放软件，等视频软件运行起来之后，可以关掉此python程序。不关闭也可以。
```python
import time
import pyperclip

def clipboard_monitor():
    last_value = '1'
    while True:
        current_value = pyperclip.paste()
        if current_value == '':
            pyperclip.copy(last_value)
        else:
            last_value = current_value

if __name__ == "__main__":
    clipboard_monitor()
```
之后剪贴板恢复正常使用。
