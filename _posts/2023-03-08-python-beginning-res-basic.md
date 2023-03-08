---
layout:     post
title:      python-step-by-step
subtitle:   basic concept
date:       2023-03-08
author:     加菲猫
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - python
    - step-by-step
    - basic
---


## Syntax and semantics ##

### Indentation ###
空格缩进取代大括号，代码显得非常简洁，代码的层级关系通过空格缩进来体现。

### naming conventions ###
命名约定：

- 类名第一个字母大写，其他标识符第一个字符全部小写
- 以一个_开始的为私有变量
- 以两个_开始为强私有变量
- 以两个_结尾的为系统特定名字

### Reserved Words ###
保留字：
**and exec not as finally or assert for pass break from print class global raise continue if return def import try del in while elif is with else lambda yield except**

### quotation ###

python接收单引号，双引号，三引号，都是用来表示字符串
   
	word = 'word'
	sentence = "This is a sentence."
	paragraph = """This is a paragraph"""

### comments ###

注释：暂时只有单行注释，可以单独注释或者注释在末尾
    
    #first comment
	x = 'hello'  #second comment

### input ###

跟C中类似

    import sys

	x = input("input the file name\n")
	print(x)

	file = open(x, 'w')
	print(file)

输入文件路径后，得到的运行结果为
    
	input the file name
	d:\\testg.gz
	d:\\testg.gz
	<_io.TextIOWrapper name='d:\\\\testg.gz' mode='w' encoding='cp936'>

### suite ###

suite:块，在class，def，if，while等后面，以关键字开始，：结尾标志一个块
    
    if 0 == '0':
    	print("is true")
	else : #end with ：
    	print("is false")

### standard data type ###

#### Numers ####

Numbers分为支持三种数据格式数据：

	- int类型（signed integers） *10,-786,080*
	- float(floating point real values) *11.10 32.3+e18*
	- complex(complex numbers)  *3.14j*

#### String ####

string操作类型有如下 

    str = 'Hello World!'

	print (str)          # 整个字符串
	print (str[0])       # 返回字符串的第1个元素
	print (str[2:4])     # 返回字符串的第3个到第4个元素
	print (str[2:])      # 返回字符串的第3个到最后的元素
	print (str * 2)      # Prints string two times
	print (str + "TEST") # Prints concatenated string


#### List ####

list类似于c语言的数组，但是list元素可以是不同类型的，list操作跟string操作基本一致

list的基本操作如下

    list = [ 'abcd', 786 , 2.23, 'john', 70.2 ]
	tinylist = [123, 'john']

	print (list)          # Prints complete list
	print (list[0])       # Prints first element of the list
	print (list[1:3])     # Prints elements starting from 2nd till 3rd 
	print (list[2:])      # Prints elements starting from 3rd element
	print (tinylist * 2)  # Prints list two times
	print (list + tinylist) # Prints concatenated lists

#### tuple ####

元组除了使用语法跟list不一样外，还有个重要特性只读，不能修改，可以看做是只读的list。

tuple的基本操作如下：

    tuple = ( 'abcd', 786 , 2.23, 'john', 70.2  )
	tinytuple = (123, 'john')

	print (tuple)           # Prints complete tuple
	print (tuple[0])        # Prints first element of the tuple
	print (tuple[1:3])      # Prints elements starting from 2nd till 3rd 
	print (tuple[2:])       # Prints elements starting from 3rd element
	print (tinytuple * 2)   # Prints tuple two times
	print (tuple + tinytuple) # Prints concatenated tuple

#### dictionary ####

字典类型的跟map的key/value键值对比较一致，

dictionary的基本操作：

    dict = {}
	dict['one'] = "This is one"
	dict[2]     = "This is two"

	tinydict = {'name': 'john','code':6734, 'dept': 'sales'}


	print (dict['one'])       # Prints value for 'one' key
	print (dict[2])           # Prints value for 2 key
	print (tinydict)          # Prints complete dictionary
	print (tinydict.keys())   # Prints all the keys
	print (tinydict.values()) # Prints all the values

### data type conversion ###

类型转换，python的类型转换很简单，直接用类型作为方法名，传参即可。

#### int（x [, base]） ####

    x = int("123")
	print(x)

#### float ####

	x = float("123")
	print(x)

#### complex(real [,image])

	x = complex(2,3)
	print(x)
    实部为2，虚部为3的复数

#### ####

## operators ##

操作符分为：

### Arichmetic Operators ###

常见的算术运算
 
    x = 10
	y = 21
	
	x + y = 31
	x - y = -11
	x * y  = 210
	y / x = 2.1
	y % x  = 1
	x ** y  = 10的21次幂
	y // x = 2  9//2 = 4 -11/3 = -4

### Comparison Operator ###

常见的比较运算

    10 == 21 false
	10 ！= 21 true
    10 > 21 false
	10 < 21 true
	10 >= 21 false
	10 <= 21 true

### assignment operator ###

常见的赋值运算：

    c =  a + b 
	c += a
	c -= a
	c *= a
	c /= a 整除
	c %= a 求余
	c **= a 指数
    c //= a floor

### bitwise operators ###

常见的位运算

    a = 0011 1100 (60)
	b = 0000 1101 (13)
	
	a & b = 00000 1100 按位与
	a | b = 0011 1101 按位或
	a ^ b  = 0011 0001 按位异或
	~a = 1100 0011 取反
	a << 2 = 1111 0000 (240)
	a >> 2 = 0000 1111 (15)

### logical operator ###

常见的逻辑运算：

    a = true 
	b = false

	a and  b is false
	a or b is true
	not a is false

### 成员运算 ###

常见的成员运算：

    a = 10
	b = 20
	list = [1, 2, 3, 4, 5 ]

	if ( a in list ):
  		print ("Line 1 - a is available in the given list")
	else:
   		print ("Line 1 - a is not available in the given list")

	if ( b not in list ):
   		print ("Line 2 - b is not available in the given list")
	else:
   		print ("Line 2 - b is available in the given list")

	c=b/a
	if ( c in list ):
   		print ("Line 3 - a is available in the given list")
	else:
   		print ("Line 3 - a is not available in the given list")

### identity operator ###

对象实体测试：比较两个对象的内存地址是否一致

    a = 20
	b = 20
	print ('Line 1','a=',a,':',id(a), 'b=',b,':',id(b))

	if ( a is b ):
  		print ("Line 2 - a and b have same identity")
	else:
   		print ("Line 2 - a and b do not have same identity")

	if ( id(a) == id(b) ):
   		print ("Line 3 - a and b have same identity")
	else:
   		print ("Line 3 - a and b do not have same identity")

	b = 30
		print ('Line 4','a=',a,':',id(a), 'b=',b,':',id(b))

	if ( a is not b ):
   		print ("Line 5 - a and b do not have same identity")
	else:
   		print ("Line 5 - a and b have same identity")

	运行结果：
	Line 1 a= 20 : 1361467424 b= 20 : 1361467424
	Line 2 - a and b have same identity
	Line 3 - a and b have same identity
	Line 4 a= 20 : 1361467424 b= 30 : 1361467584
	Line 5 - a and b do not have same identity

## condition statement ##

	python 将0或者null值当做false，其他都为true
	if
	if ... else ...
	if (if ... else ... ) else ...

## loops ##

循环语句：
	while loop  while循环
	for loop for循环

	break 语句
	continue 语句
	pass 语句
	
	#例子，iterator对象的iter与next方法混合使用
    list = [1,2,3,4]
	it = iter(list)
	
	while true
		try:
			print（next（it））
		except StopIteration:
			sys.exit()

## Numbers ##

### Type Conversion ###
类型转换

    int（x）   int（13.0）
	long（x）  long（12）  python3中舍弃了long类型
	float（x） float（12）
	complex(x[,y]) complex(1,2)-> 1+2j 复数形式在实际应用中很少使用

### Mathematical Functions ###
数学函数

    abs（x） 绝对值
	math.ceil(x) 返回比当前值大的最小整数
	math.exp(x)  e的x次方
	math.fabs(x) 
	math.floor(x) 地板值
	math.log(x)
	math.log10(x)
	math.max(x1,x2,...) 最大值
	math.min(x1,x2,...) 最小值
	math.modf(x) math.modf(math.pi) 将数值分解为整数与小数部分
	math.pow(x,y)  x的y次方
	round(x[,n])  四舍五入
	math.sqrt(x) 平方根
	...

### random function ###
随机函数

    random.choice(seq) random.choice([1,2,3,4]) 随机返回一个数值
	randrange（[start,]stop[,step]) randrange(0,10,2) 以2为步调
	random.random()  返回0<=r<1的随机值
	random.seed([x])
	random.shuffle(list) 洗牌
	random.uniform(x,y)

## function ##
函数

python中函数参数都是传址而不是传值

函数参数类型有四种类型：
	
- Required argument -
- Keyword argument -带关键字参数
- Default argument -默认参数
- Variable-length argument -可变参数

### lambda函数 ###

lambda语法：只允许单行表达式
> lambda [arg1 [,arg2,.....argn]]:expression

    sum = lambda arg1, arg2: arg1 + arg2


	# Now you can call sum as a function
	print ("Value of total : ", sum( 10, 20 ))
	print ("Value of total : ", sum( 20, 20 ))

### global vs local variables ###

全局与局部变量：方法外测的都是全局变量，方法内部的都是局部变量

    total = 0 # This is global variable.
	# Function definition is here
	def sum( arg1, arg2 ):
   		# Add both the parameters and return them."
   		total = arg1 + arg2; # Here total is local variable.
   		print ("Inside the function local total : ", total)
   		return total

	# Now you can call sum function
	sum( 10, 20 )
	print ("Outside the function global total : ", total )

	#result：
	#Inside the function local total :  30
	#Outside the function global total :  0

## modules ##

modules：python核心设计

## Exception ##

异常部分分为：
1. Exception Handling
2. Assertions

常见的异常：
1. Excepiton -全部的异常
2. StopIteration —停止迭代异常
3. SystemExit —调用sys.exit()触发异常
4. StandardError —除了StopIteration与SystemExit后的其他异常
5. ArithmeticError -所有的数值计算
6. OverflowError —计算溢出异常
7. FloatingPointError —浮点数异常
8. ZeroDivisionError -0除异常
9. AssertionError -断言异常
10. AttributeError -属性异常
11. EOFError - input无输入或者到达文件尾时触发
12. ImportError -导入失败
13. KeyboardInterrupt -执行打断
14. LookupError —查找失败异常
15. KeyError —在dict中没有找到指定的键值
16. NameError -变量没有找到
17. UnboundLocalError -使用未赋值的本地变量
18. EnvironmentError -环境异常
19. IOError -IO异常
20. OSError -操作系统相关的异常
21. SyntaxError -语法异常
22. IndentationError -缩进异常
23. SystemError -系统异常
24. SystemExit -
25. TypeError -类型异常
26. ValueError
27. RuntimeError
28. NotImplementedError -之类没有实例化父类的抽象方法

python脚本的异常必须捕获，否则程序会立即终止退出
python使用 try：。。。except xx： 。。。 else： 。。。语法

    try:
   		fh = open("testfile", "w")
   		fh.write("This is my test file for exception handling!!")
	except IOError:
   		print ("Error: can\'t find file or read data")
	else:
   		print ("Written content in the file successfully")
   		fh.close()

    #try finally
	try:
    	fh = open("testfile", "w")
    	try:
        	fh.write("This is my test file for exception handling!!")
    	finally:
        	print ("Going to close the file")
        	fh.close()
	except IOError:
    	print ("Error: can\'t find file or read data")