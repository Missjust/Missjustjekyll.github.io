---
layout: post
title: 新闻爬虫展示网站搭建
subtitle: 怎么搭建的呢
# cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
# tags: [books, test]
---

这篇文章介绍了搭建一个新闻爬虫展示网站的详细步骤，包括数据库创建、后端功能接口实现和前端界面的设计。

网站首页

![pic1]({{ "/assets/img/20210607203201807.png" | absolute_url }})

# 一、数据库修改

数据库里新增了用户表、日志表

用户表属性包括用户id（主键），用户名（唯一），密码，权限（默认为true，即用户有查看网站数据的权限）

日志表属性包括日志id（主键），用户名，用户操作，操作时间

```sql
CREATE TABLE users (
   id_users serial PRIMARY key,
   username varchar(255) not null UNIQUE,
   password varchar(255) not null,
   power boolean not null DEFAULT TRUE
);

CREATE TABLE logs(
   id_logs serial PRIMARY KEY,
   username varchar(255) not null,
   action varchar(255) not null,
   operation_time TIMESTAMP  DEFAULT CURRENT_TIMESTAMP
);
```

# 二、爬虫数据查询

采用bootstrap框架的表单样式，以及bootstrapTable展示查询结果，查询结果可以分页显示，同时，从数据库的news表中查询数据时按照新闻发表时间降序排列

![pic2]({{ "/assets/img/20210607205840306.png" | absolute_url }})

# 三、爬虫数据图表显示

1.下拉列表选择需要展示的图表：包括折线图、饼图、词云（echarts）

2.词云图展示了新闻关键词中出现频率较高的词汇，从图中可以看出是疫苗、中国、2021。

绘制词云时展示了数量前500的关键词

```javascript
router.get('/get_date_num', function (req, res) {
    var username = req.cookies.username;
    res.writeHead(200, { 'Content-Type': 'text/html;charset=utf-8' });
    var fetchSql = "SELECT publish_date,count(*) FROM news group by publish_date order by publish_date;";
    pgsql.query_noparam(fetchSql, function (err, result, fields) {
        var fetchAddSql = 'INSERT INTO logs (username, action)' + 'VALUES ($1, $2);';
        var fetchAddSql_Params = [username, 'view chart'];
        pgsql.query(fetchAddSql, fetchAddSql_Params, function(err,result) {
            if(err) {
                console.log(err);
            }
        })
        // console.log(result.rows);
        res.end(JSON.stringify(result.rows));
        // console.log(res);
    });
});
```
![pic3]({{ "/assets/img/20210607211003598.png" | absolute_url }})

# 四、扩展功能

1.实现对爬虫数据中文分词的查询（第一个项目已经实现，采用nodejieba对新闻内容进行分词，去除停用词后，构建关键词表，实现查询）

2.用Elastic Search+Kibana展示爬虫的数据结果

步骤：

安装jdk

安装解压  Elasticsearch、kibana

启动Elasticsearch 运行bin目录下bat文件

启动kibana 运行bin目录下bat文件

访问Elasticsearch   http://localhost:9200/   

访问kibana  http://localhost:5601/

将postgresql数据库中的新闻表和关键词表导出为csv文件，再导入kibana

利用kibana上的图例绘制图表

利用kibana的dashboard展示绘制的图表

绘制了新闻发表数和新闻发布时间的水平条形图、关键词的词云图（展示数目最多的100个词）、来源网站新闻数的环形图、新闻发表数和新闻发布时间的垂直面积图

![pic4]({{ "/assets/img/20210608194809448.png" | absolute_url }})