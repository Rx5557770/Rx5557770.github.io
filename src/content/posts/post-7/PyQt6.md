
---
title: PyQt6
published: 2025-10-23
description: ''
image: ''
tags: ['code', 'python', 'PyQt6']
category: 代码日常
draft: false
lang: zh-CN
---
    ### 基本结构

> 从PyQt6.QtWidgets 中引用一些依赖与控件显示到页面。

```python
import sys  
from PyQt6.QtWidgets import QApplication, QWidget, QLabel  
if __name__ == '__main__':  
    app = QApplication(sys.argv)  
    w = QWidget()  # 主视窗  
    w.setWindowTitle('测试')  
    w.resize(300, 200)  # 设置视图大小  
  
    label = QLabel(w)  # 添加一个label  
    label.setText('标签')  
      
  
    w.show()  # 显示视窗  
    sys.exit(app.exec())
```

也可以使用类的方式调用

```python
import sys  
from PyQt6.QtWidgets import QApplication, QWidget, QLabel  
  
class myWidget(QWidget):  
    def __init__(self):  
        super().__init__()  
        self.setWindowTitle('测试')  
        self.resize(300, 200)  
          
        self.ui()  # 调用布局  
    def ui(self):  
        self.label = QLabel(self)  
        self.label.setText('标签')  
  
if __name__ == '__main__':  
    app = QApplication(sys.argv)  
    w = myWidget()  
    w.show()  # 显示视窗  
    sys.exit(app.exec())
```

