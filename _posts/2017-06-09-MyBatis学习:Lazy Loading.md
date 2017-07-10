---
layout: post_layout
title: MyBatis学习:Lazy Loading
time: 2017年6月9日
location: 北京
pulished: true
excerpt_separator: "##"
---

## 一.背景
最近在阅读项目的Persist层代码，使用的是MyBatis框架，在读到Mapper文件时，发现了一个N+1查询问题。
概括地讲,N+1 查询问题是这样的:
>1. 你执行了一个单独的 SQL 语句来获取结果列表(就是“+1”)。
2. 对返回的每条记录,你执行了一个查询语句来为每个加载细节(就是“N”)。

这个问题可能会导致成百上千的 SQL 语句被执行。
比如在Blog类中，有一个List<Post>，那么在查询Blog时，也会从post表中查询。在MyBatis中，通常有两种写法:
### 1.嵌套查询
```
<resultMap id="blogResult" type="Blog">
  <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>
 
<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>
 
<select id="selectPostsForBlog" resultType="Post">
  SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>
```
这种写法下，每次selectBlog会产生多次select操作：先从blog表中得到满足条件的ResultSet，然后依次在post表中查询获得最终结果。

### 2.嵌套结果
```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <result property="body" column="post_body"/>
  </collection>
</resultMap>
  
<select id="selectBlog" resultMap="blogResult">
  select
  B.id as blog_id,
  B.title as blog_title,
  B.author_id as blog_author_id,
  P.id as post_id,
  P.subject as post_subject,
  P.body as post_body,
  from Blog B
  left outer join Post P on B.id = P.blog_id
  where B.id = #{id}
</select>
```
使用了join将两个表连接起来，只会产生一次查询，但可维护性，可读性，复用性较第一种低。

### 二.探索
频繁的查询操作是很耗时的，但是第二种方法的可维护性和复用性又比较低。而在我们的项目中使用了第一种方法。
那怎么减缓N+1问题带来的性能问题呢？阅读项目MyBatis配置文件，发现使用了Lazy Loding机制。
```
<setting name="lazyLoadingEnabled" value="true"/>
```
LazyLoad 的作用：在数据与对象进行 mapping 操作时，只有在真正使用到该对象时，才进行 mapping 操作，以减少数据库查询开销，从而提升系统性能。
为此我进行了测试，测试代码：
```java
@Test
public void testFindOrderAndCustomer() {
    List<Order> orders = orderAndCustomerMapper.findOrderAndCustomer();
     
    for(Order order : orders) {
        System.out.println(order.toString());
    }
 
}
```
不开启lazyLoading:
![图片](http://7xlv11.com1.z0.glb.clouddn.com/mybatis1.png)
开启lazyLoding:
![图片](http://7xlv11.com1.z0.glb.clouddn.com/mybatis2.png)
很明显在使用到对象时，才从数据库中进行查询。
## 三.Lazy Loading原理
MyBatis主要通过cglib（默认）和Javassist代理实现Lazy Loding。下图是一个cglib实现的流程图:（来自[mybatis源码学习--mybatis懒加载内部原理]( http://blog.csdn.net/mingtian625/article/details/47358003))
![图片](http://7xlv11.com1.z0.glb.clouddn.com/mybatis3.png)
概况的讲，MyBatis会生成一个代理类，它继承了 Result 对象所属的类（被代理类）并重写了被代理类的所有的方法，在代理对象上调用它的get/set 方法时会触发Lazy Loading动作且只会触发一次。
Lazy Loding是一次查询操作，查询所需的属性、参数、 SQL 语句等相关的条件都已经封装成一个Map到代理对象内部，键是查询的属性名称，值是查询该属性所需的条件，包括参数、sql 语句等。加载完一个属性之后会把这个属性从 Map 中移除，所以再次出发Lazy Loading操作时，MyBatis 就知道该属性已经被被加载过了，不会重复加载。
关于代理的具体细节就不细说了，可以参考[CGLib实现](http://blog.csdn.net/mingtian625/article/details/47358003)和[Javassist](http://blog.csdn.net/shfqbluestone/article/details/52853460)。
## 四.思考
那是不是任何时候都用Lazy Loding，答案肯定是否定的。Lazy Loading也有缺点，因为会多次连接数据库，同时会增加数据库的压力。所以在实际使用时，会根据业务衡量是否使用延迟加载。
假设联合查询的对象全部或者大部分都会很快使用，那使用Lazy Loding肯定是不适合的，此时可以考虑使用前文所说的第二种方法:嵌套结果。

参考文章：
http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html
http://blog.csdn.net/mingtian625/article/details/47358003
http://blog.csdn.net/shfqbluestone/article/details/52853460






