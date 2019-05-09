## Python 标识符
- 在 Python 里，标识符由字母、数字、下划线组成。
- 在 Python 中，所有标识符可以包括英文、数字以及下划线(_)，但不能以数字开头。
- Python 中的标识符是区分大小写的。
- 以下划线开头的标识符是有特殊意义的。以单下划线开头 _foo 的代表**不能直接访问**的类属性，需通过类提供的接口进行访问，不能用 from xxx import * 而导入。
- 以双下划线开头的 __foo 代表类的私有成员，以双下划线开头和结尾的   代表 Python 里特殊方法专用的标识，如 __ _init__ _() 代表类的构造函数。

## Python 保留字符
| | | | 
| :--------:| :-----: | :----: |
| and     	| exec	  | not    |
| assert   	| finally | or     | 
| break     | for     | pass   | 
| class   	| from	  | print  | 
| continue	| global  | raise  | 
| def      	| if	    | return | 
| del     	| import  | try    | 
| elif	    | in	    | while  | 
| else	    | is	    | with   | 
| except	  | lambda  | yield  | 

## 行和缩进
Python 与其他语言最大的区别就是，Python 的代码块不使用大括号 {} 来控制类，函数以及其他逻辑判断。python 最具特色的就是用缩进来写模块。

## 多行语句
Python语句中一般以新行作为语句的结束符。但是我们可以使用斜杠（ \）将一行的语句分为多行显示，如下所示：
```java
total = item_one + \
        item_two + \
        item_three
```
语句中包含 [], {} 或 () 括号就不需要使用多行连接符。如下实例：
```java
days = ['Monday', 'Tuesday', 'Wednesday',
        'Thursday', 'Friday']
```
## Python 引号
Python 可以使用引号( ' )、双引号( " )、三引号( ''' 或 """ ) 来表示字符串，引号的开始与结束必须的相同类型的。其中三引号可以由多行组成，编写多行文本的快捷语法，常用于文档字符串，在文件的特定地点，被当做注释。
```Python 
word = 'word'
sentence = "这是一个句子。"
paragraph = """这是一个段落。
包含了多个语句"""
```

```Python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 文件名：test.py


'''
这是多行注释，使用单引号。
这是多行注释，使用单引号。
这是多行注释，使用单引号。
'''

"""
这是多行注释，使用双引号。
这是多行注释，使用双引号。
这是多行注释，使用双引号。
"""
```
