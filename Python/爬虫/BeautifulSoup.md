---
title: BeautifulSoup
---



# 一、BS4

## 1.1 安装

```python
#安装 Beautiful Soup
pip install bs4

#安装解析器
Beautiful Soup支持Python标准库中的HTML解析器,还支持一些第三方的解析器,其中一个是 lxml .根据操作系统不同,可以选择下列方法来安装lxml:

$ apt-get install Python-lxml

$ easy_install lxml

$ pip install lxml

另一个可供选择的解析器是纯Python实现的 html5lib , html5lib的解析方式与浏览器相同,可以选择下列方法来安装html5lib:

$ apt-get install Python-html5lib

$ easy_install html5lib

$ pip install html5lib
```

> **下表列出了主要的解析器,以及它们的优缺点,官网推荐使用lxml作为解析器,因为效率更高. 在Python2.7.3之前的版本和Python3中3.2.2之前的版本,必须安装lxml或html5lib, 因为那些Python版本的标准库中内置的HTML解析方法不够稳定.**


| 解析器           | 使用方法                                                     | 优势                                                  | 劣势                                            |
| ---------------- | ------------------------------------------------------------ | ----------------------------------------------------- | ----------------------------------------------- |
| Python标准库     | `BeautifulSoup(markup, "html.parser")`                       | Python的内置标准库执行速度适中文档容错能力强          | Python 2.7.3 or 3.2.2)前 的版本中文档容错能力差 |
| lxml HTML 解析器 | `BeautifulSoup(markup, "lxml")`                              | 速度快文档容错能力强                                  | 需要安装C语言库                                 |
| lxml XML 解析器  | `BeautifulSoup(markup, ["lxml", "xml"])``BeautifulSoup(markup, "xml")` | 速度快唯一支持XML的解析器                             | 需要安装C语言库                                 |
| html5lib         | `BeautifulSoup(markup, "html5lib")`                          | 最好的容错性以浏览器的方式解析文档生成HTML5格式的文档 | 速度慢不依赖外部扩展                            |

## 1.2 基本使用

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

#基本使用：容错处理,文档的容错能力指的是在html代码不完整的情况下,使用该模块可以识别该错误。使用BeautifulSoup解析上述代码,能够得到一个 BeautifulSoup 的对象,并能按照标准的缩进格式的结构输出
from bs4 import BeautifulSoup
soup=BeautifulSoup(html_doc,'lxml')  # 这两种都可以
soup=BeautifulSoup(open("xx.xxx"),'lxml') # 这两种都可以
res=soup.prettify() # 处理好缩进，结构化显示
print(res)
```

## 1.3 选择器

### 1.3.1 标签选择器

> 标签选择器其实就是在遍历文档树：从根往下找，即直接通过标签名字选择，特点是速度快，但如果存在多个相同的标签则只返回第一个

```python
- 获取标签
	- 获取标签的名称
	- 获取标签的属性
	- 获取标签的内容
- 嵌套选择
- 子节点、子孙节点
- 父节点、祖先节点
- 兄弟节点
```

**栗子**

```python

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p id="my p" class="title">ooo<b id="bbb" class="boldest">The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

#1、用法
from bs4 import BeautifulSoup
soup=BeautifulSoup(html_doc,'lxml')
# soup=BeautifulSoup(open('a.html'),'lxml')

print(soup.p) #存在多个相同的标签则只返回第一个
print(soup.a) #存在多个相同的标签则只返回第一个

#2、获取标签的名称
print(soup.p.name)

#3、获取标签的属性
print(soup.p.attrs)

#4、获取标签的内容
print(soup.p.string) # p下必须只有文本，没有标签，才能取到，否则为None
print(soup.p.strings) #text的生成器版，拿到一个生成器对象, 取到p下所有的文本内容
print(soup.p.text) # 取p下所有的文本内容
print(soup.p.get_text())
for line in soup.stripped_strings: #去掉空白
    print(line)


'''
如果tag包含了多个子节点,tag就无法确定 .string 方法应该调用哪个子节点的内容, .string 的输出结果是 None，如果只有一个子节点那么就输出该子节点的文本，比如下面的这种结构，soup.p.string 返回为None,但soup.p.strings就可以找到所有文本
<p id='list-1'>
    哈哈哈哈
    <a class='sss'>
        <span>
            <h1>aaaa</h1>
        </span>
    </a>
    <b>bbbbb</b>
</p>
'''

#5、嵌套选择
print(soup.head.title.string)
print(soup.body.a.string)


#6、子节点、子孙节点
print(soup.p.contents) #p下所有子节点
print(soup.p.children) #得到一个迭代器,包含p下所有子节点

for i,child in enumerate(soup.p.children):
    print(i,child)

print(soup.p.descendants) #获取子孙节点,p下所有的标签都会选择出来
for i,child in enumerate(soup.p.descendants):
    print(i,child)

#7、父节点、祖先节点
print(soup.a.parent) #获取a标签的父节点
print(soup.a.parents) #找到a标签所有的祖先节点，父亲的父亲，父亲的父亲的父亲...


#8、兄弟节点
print('=====>')
print(soup.a.next_sibling) #下一个兄弟
print(soup.a.previous_sibling) #上一个兄弟

print(list(soup.a.next_siblings)) #下面的兄弟们=>生成器对象
print(soup.a.previous_siblings) #上面的兄弟们=>生成器对象
```

### 1.3.2 find() 和 find_all()

> find() 和 find_all()其实就是在搜索文档树：BeautifulSoup定义了很多搜索方法,这里着重介绍这两个，其它方法的参数和用法类似
>
> 其他方法：https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#find-parents-find-parent

**知识储备**

```python
# 字符串、正则表达式、列表、True、方法
# 这五种过滤器是说：find(name,attrs,recursive,text,**kwargs)和find_all(name,attrs,recursive,text,**kwargs)的参数的值是这五种中任何一种组成
# 比如name的值可以是这五种中任何一种组成
```

**find_all**

> find_all( name , attrs , recursive , text , **kwargs )

```python
# name: 标签名，name参数的值可以是任一类型的过滤器：字符串,正则表达式,列表,方法或是 True 
print(soup.find_all(name=re.compile('^t'))) # 查找t开头的标签
print(soup.find_all(name='a') # 查找a标签

# attrs：根据属性查找标签
print(soup.find_all('p',attrs={'class':'story'})) # 查找class为story的p标签


# recursive：调用tag的find_all() 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 recursive=False .
print(soup.html.find_all('a'))
print(soup.html.find_all('a',recursive=False))  
  
  
# text: 值可以是：字符，列表，True，正则
print(soup.find_all(text='Elsie'))
print(soup.find_all('a',text='Elsie'))
  
  
  
# **kwargs：key=value的形式，value可以是过滤器：字符串 , 正则表达式 , 列表, True，这个有点鸡肋，我们使用attrs就行了，而且使用attrs，查找class时，还不需要在后面加"_"
print(soup.find_all(id=re.compile('my')))
print(soup.find_all(href=re.compile('lacie'),id=re.compile('\d')))
print(soup.find_all(id=True)) #查找有id属性的标签
print(soup.find_all('a',class_='sister')) #查找类为sister的a标签
  
  
"""
有些tag属性在搜索不能使用,比如HTML5中的 data-* 属性:
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>','lxml')
data_soup.find_all(data-foo="value") 
报错：SyntaxError: keyword can't be an expression
但是可以通过 find_all() 方法的 attrs 参数定义一个字典参数来搜索包含特殊属性的tag:
print(data_soup.find_all(attrs={"data-foo": "value"}))
"""

  
  
# limit参数:如果文档树很大那么搜索会很慢.如果我们不需要全部结果,可以使用 limit 参数限制返回结果的数量.效果与SQL中的limit关键字类似,当搜索到的结果数量达到 limit 的限制时,就停止搜索返回结果
print(soup.find_all('a',limit=2))



# 如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数 ,如果这个方法返回 True 表示当前元素匹配并且被找到,如果不是则反回 False
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')

print(soup.find_all(has_class_but_no_id))

  
  
'''
像调用 find_all() 一样调用tag
find_all() 几乎是Beautiful Soup中最常用的搜索方法,所以我们定义了它的简写方法. BeautifulSoup 对象和 tag 对象可以被当作一个方法来使用,这个方法的执行结果与调用这个对象的 find_all() 方法相同,下面两行代码是等价的:
soup.find_all("a")
soup("a")
这两行代码也是等价的:
soup.title.find_all(text=True)
soup.title(text=True)
'''

```

**find**

```python
#3、find( name , attrs , recursive , text , **kwargs )

find_all() 方法将返回文档中符合条件的所有tag,尽管有时候我们只想得到一个结果.比如文档中只有一个<body>标签,那么使用 find_all() 方法来查找<body>标签就不太合适, 使用 find_all 方法并设置 limit=1 参数不如直接使用 find() 方法.下面两行代码是等价的:

soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]
soup.find('title')
# <title>The Dormouse's story</title>

唯一的区别是 find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果.
find_all() 方法没有找到目标是返回空列表, find() 方法找不到目标时,返回 None .
print(soup.find("nosuchtag"))
# None

soup.head.title 是 tag的名字方法的简写.这个简写的原理就是多次调用当前tag的find() 方法:

soup.head.title
# <title>The Dormouse's story</title>
soup.find("head").find("title")
# <title>The Dormouse's story</title>
```

### 1.3.3 css选择器

```python
#该模块提供了select方法来支持css,详见官网:https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id37
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title">
    <b>The Dormouse's story</b>
    Once upon a time there were three little sisters; and their names were
    <a href="http://example.com/elsie" class="sister" id="link1">
        <span>Elsie</span>
    </a>
    <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
    <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
    <div class='panel-1'>
        <ul class='list' id='list-1'>
            <li class='element'>Foo</li>
            <li class='element'>Bar</li>
            <li class='element'>Jay</li>
        </ul>
        <ul class='list list-small' id='list-2'>
            <li class='element'><h1 class='yyyy'>Foo</h1></li>
            <li class='element xxx'>Bar</li>
            <li class='element'>Jay</li>
        </ul>
    </div>
    and they lived at the bottom of a well.
</p>
<p class="story">...</p>
"""
from bs4 import BeautifulSoup
soup=BeautifulSoup(html_doc,'lxml')

#1、CSS选择器
print(soup.p.select('.sister'))
print(soup.select('.sister span')) # .sister下面的子子孙孙的span标签
print(soup.select('.sister>span')) # .sister下面所有儿子节点是span的标签

print(soup.select('#link1'))
print(soup.select('#link1 span'))

print(soup.select('#list-2 .element.xxx'))

print(soup.select('#list-2')[0].select('.element')) #可以一直select,但其实没必要,一条select就可以了

# 2、获取属性
print(soup.select('#list-2 h1')[0].attrs)

# 3、获取内容
print(soup.select('#list-2 h1')[0].get_text())
```

## 1.6 修改文档树

> https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id40

## 1.7 总结

```python
# 总结:
#1、推荐使用lxml解析库
#2、讲了三种选择器:标签选择器,find与find_all，css选择器
    1、标签选择器筛选功能弱,但是速度快
    2、建议使用find,find_all查询匹配单个结果或者多个结果
    3、如果对css选择器非常熟悉建议使用select
#3、记住常用的获取属性attrs和文本值get_text()的方法