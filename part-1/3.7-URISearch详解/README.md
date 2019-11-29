# URI Search详解
## 课程Demo
```

#基本查询
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s

#接收时间 rts 在两天以内的事件
GET /event/_search?q=rts:>=now-2d

#带profile
GET /movies/_search?q=2012&df=title
{
	"profile":"true"
}


#泛查询，正对_all,所有字段
GET /movies/_search?q=2012
{
	"profile":"true"
}

#指定title字段
GET /movies/_search?q=title:2012&sort=year:desc&from=0&size=10&timeout=1s
{
	"profile":"true"
}


# 查找美丽心灵, Mind为泛查询
GET /movies/_search?q=title:Beautiful Mind
{
	"profile":"true"
}

# 泛查询
GET /movies/_search?q=title:2012
{
	"profile":"true"
}

#使用引号，Phrase查询 (包含 Beautiful 和 Mind,切顺序保持一致 )
GET /movies/_search?q=title:"Beautiful Mind"
{
	"profile":"true"
}

#分组，Term查询 ，等效于 (Beautiful OR Mind)
GET /movies/_search?q=title:(Beautiful Mind)
{
	"profile":"true"
}


#布尔操作符
# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful AND Mind)
{
	"profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful NOT Mind)
{
	"profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful %2BMind)
{
	"profile":"true"
}


#范围查询 ,区间写法
GET /movies/_search?q=title:beautiful AND year:[2002 TO 2018%7D
{
	"profile":"true"
}


#通配符查询
GET /movies/_search?q=title:b*
{
	"profile":"true"
}

# Fuzziness (模糊匹配)
# https://blog.csdn.net/lhkaikai/article/details/25186255
# 莱文斯坦距离是一种两个字符串序列的距离度量。形式化地说，两个单词的莱文斯坦距离是一个单词变成另一个单词要求的最
# 少单个字符编辑数量（如：删除、插入和替换）
GET /movies/_search?q=title:beautifl~1
{
	"profile":"true"
}

# Proximity searches (近似度匹配)
# 近似度匹配允许指定的单词之间有间隔或以不同的顺序排列。就像模糊匹配可以 为单词中的字符指定最大编辑距离一样，
# 近似度匹配允许我们为短语中的单词指定最大编辑距离。
GET /movies/_search?q=title:"Lord Rings"~2
{
	"profile":"true"
}


```


## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/7.4/search-uri-request.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.4/search-search.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.4/query-dsl-query-string-query.html
