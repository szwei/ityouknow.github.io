---
layout: post
title: Springboot  中 Elasticsearch  使用
category: springboot
tags: [springboot]
---


*本文内容参考自： https://gitee.com/macrozheng/mall*

*如有侵权，请联系作者删除*



项目中所使用代码已开源 : https://gitee.com/szwei/elasticsearch

项目中使用依赖版本：

| 依赖          | 版本                 |
| ------------- | -------------------- |
| spring-boot   | 2.3.1.RELEASE        |
| elasticsearch | 7.9.3-windows-x86_64 |
| kibana        | 7.8.0-windows-x86_64 |

## 一、介绍

>**回忆时光**
>
>许多年前，一个刚结婚的名叫 Shay Banon 的失业开发者，跟着他的妻子去了伦敦，他的妻子在那里学习厨师。 在寻找一个赚钱的工作的时候，为了给他的妻子做一个食谱搜索引擎，他开始使用 Lucene 的一个早期版本。
>
>直接使用 Lucene 是很难的，因此 Shay 开始做一个抽象层，Java 开发者使用它可以很简单的给他们的程序添加搜索功能。 他发布了他的第一个开源项目 Compass。
>
>后来 Shay 获得了一份工作，主要是高性能，分布式环境下的内存数据网格。这个对于高性能，实时，分布式搜索引擎的需求尤为突出， 他决定重写 Compass，把它变为一个独立的服务并取名 Elasticsearch。
>
>第一个公开版本在2010年2月发布，从此以后，Elasticsearch 已经成为了 Github 上最活跃的项目之一，他拥有超过300名 contributors(目前736名 contributors )。 一家公司已经开始围绕 Elasticsearch 提供商业服务，并开发新的特性，但是，Elasticsearch 将永远开源并对所有人可用。
>
>据说，Shay 的妻子还在等着她的食谱搜索引擎…

不得不说，Elasticsearch 的作者是一个很幽默的人，大概这就是大佬普遍的特性吧，随手一写，就可以开发出足以影响世界的代码。

ES 的特性：

- 一个分布式的实时文档存储，*每个字段* 可以被索引与搜索
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

## 二、安装

==注意：== Elasticsearch  的版本和 springboot 版本 和 Spring Data Elasticsearch ==版本需要对应==，否则会有各种各样未知错误

|                  Spring Data Release Train                   |                  Spring Data Elasticsearch                   | Elasticsearch |                         Spring Boot                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :-----------: | :----------------------------------------------------------: |
| 2020.0.0[[1](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_1)] | 4.1.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_1)] |     7.9.3     | 2.4.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_1)] |
|                           Neumann                            |                            4.0.x                             |     7.6.2     |                            2.3.x                             |
|                            Moore                             |                            3.2.x                             |    6.8.12     |                            2.2.x                             |
|                           Lovelace                           |                            3.1.x                             |     6.2.2     |                            2.1.x                             |
| Kay[[2](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_2)] | 3.0.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_2)] |     5.5.0     | 2.0.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_2)] |
| Ingalls[[2](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_2)] | 2.1.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_2)] |     2.4.0     | 1.5.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.2/reference/html/#_footnotedef_2)] |

### 1. 安装 Elasticsearch 

​	下载地址：

​	[官网下载地址](https://www.elastic.co/cn/downloads/elasticsearch)（不推荐，较慢）

​	[华为镜像加速](https://mirrors.huaweicloud.com/elasticsearch/)（推荐，速度 嗖嗖嗖~~）

 - 首先通过官网下载 需要的版本，笔者下载的是Windows 版本的，等待下载完成

 - Windows  版本下载好之后，解压文件夹，双击 bin  目录下的  elasticsearch.bat 启动 ES

 - 等待启动完成后，访问 **http://localhost:9200** 

   ![image-20201211104844790](https://gitee.com/szwei/images/raw/master/img/image-20201211104844790.png)

看到这个返回结果，就是启动完成了。

安装==中文分词器==，elasticsearch-analysis-ik

下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases

下载相对应的版本后，解压，在 elasticsearch 的安装目录，plugins 目录下新建 ik 文件夹，将解压后的文件放到里面，==重启== ES

那么现在我们还差一个可视化界面

### 2.安装 kibana

​	同样的国外的较慢，直接华为下载 =>  [华为kibana镜像](https://mirrors.huaweicloud.com/kibana/)

​	下载后，解压

​	bin 目录，kibana.bat 启动，kibana 会自动的连接我们刚刚启动的 ES 

​	看到这个页面，说明启动完成

![image-20201211105607430](https://gitee.com/szwei/images/raw/master/img/image-20201211105607430.png)

 访问 http://localhost:5601/ 即可看到页面

 我们选择 左侧的  Dev Tools 进入命令输入页面

![image-20201211105834109](https://gitee.com/szwei/images/raw/master/img/image-20201211105834109.png)

## 三、简单使用

 ###  索引操作

1. **列出所有索引**

   `GET /_cat/indices`

2. **创建新的索引**

   `PUT /mygoods`

3. **查询某个索引信息**

   `GET /mygoods`

4. **删除某个索引**

   `DELETE /mygoods`

### 文档操作

1. **添加文档**

   ```bash
   PUT /mygoods2/_doc/188
   {
     "id":"188",
     "name":"添加索引文档",
     "price":"188",
     "keywords":"添加索引文档",
     "subtitle":"添加索引文档",
     "brandId" : 7,
     "categoryId" : 5
   }
   ```

2. **删除文档**

   `DELETE /mygoods/_doc/188`

3. **简单查询**

   1. 查询单个

   `GET /mygoods/_doc/188`

   2. 查询所有

   `GET /mygoods/_search`

4. **表达式查询**

   1. 查询所有

      ```bash
      GET /mygoods/_search
      {
          "query" : {
             "match_all": {}
          }
      }
      ```

   2. 查询条件查询

      ```bash
      GET /mygoods/_search
      {
          "query" : {
              "match" : {
                  "name" : "添加索引"
              }
          }
      }
      ```

## 四、Springboot 中使用

	### 1. 引入依赖

查询Springboot 和 Spring Data Elasticsearch 版本，下载对应的 ES , 引入依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

### 2. 新建实体类

```java
package cn.couldme.es.entity;

import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import lombok.EqualsAndHashCode;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.io.Serializable;

/**
 * 商品表(Goods)表实体类
 *
 * @Author szwei
 * @Date 2020-12-06 21:45:37
 */
@Data
@TableName
@EqualsAndHashCode(callSuper = false)
@Document(indexName = "mygoods",type = "_doc", shards = 1,replicas = 0)
public class Goods implements Serializable {

    private static final long serialVersionUID = -1; 
    /**
    * 自增 => @TableId(type = IdType.AUTO)
    */
    @TableId
    private Long id;
        
    /*名称*/
    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String name;
        
    /*价格*/    
    private Double price;
        
    /*关键词*/
    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String keywords;
        
    /*标题*/
    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String subtitle;
        
    /*品牌商*/
    @Field(type=FieldType.Keyword)
    private Long brandId;
        
    /*分类名称*/
    @Field(type=FieldType.Keyword)
    private Long categoryId;
}
```

### 3. 继承 ElasticsearchRepository

```java
package cn.couldme.es.repository;

import cn.couldme.es.entity.Goods;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

/**
 * 搜索商品ES操作类
 * Created by macro on 2018/6/19.
 */
public interface EsGoodsRepository extends ElasticsearchRepository<Goods, Long> {
    /**
     * 搜索查询
     *
     * @param name              商品名称
     * @param keywords          商品关键字
     * @param page              分页信息
     */
    Page<Goods> findByNameOrKeywords(String name, String keywords, Pageable page);

}

```



### 4. EsGoodstService

```java
package cn.couldme.es.service;

import cn.couldme.es.entity.Goods;
import org.springframework.data.domain.Page;

import java.util.List;

/**
 * 搜索商品管理Service
 * Created by macro on 2018/6/19.
 */
public interface EsGoodstService {
    /**
     * 从数据库中导入所有商品到ES
     */
    int importAll();

    /**
     * 根据id删除商品
     */
    void delete(Long id);

    /**
     * 根据id创建商品
     */
    Goods create(Long id);

    /**
     * 批量添加商品
     */
    void batchCreate(List<Long> ids);

    /**
     * 批量删除商品
     */
    void batchDelete(List<Long> ids);

    /**
     * 根据关键字搜索名称或者副标题
     */
    Page<Goods> search(String keyword, Integer pageNum, Integer pageSize);

    /**
     * 根据关键字搜索名称或者副标题复合查询
     */
    Page<Goods> search(String keyword, Long brandId, Long categoryId, Integer pageNum, Integer pageSize, Integer sort);

    /**
     * 根据商品id推荐相关商品
     */
    Page<Goods> recommend(Long id, Integer pageNum, Integer pageSize);

}


```



### 5. EsGoodstServiceImpl

```java
@Slf4j
@Service
public class EsGoodstServiceImpl implements EsGoodstService {

    @Autowired
    private GoodsMapper goodsMapper;
    @Autowired
    private EsGoodsRepository productRepository;
    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;
    @Override
    public int importAll() {
        List<Goods> GoodsList = goodsMapper.selectList(null);
        Iterable<Goods> GoodsIterable = productRepository.saveAll(GoodsList);
        Iterator<Goods> iterator = GoodsIterable.iterator();
        int result = 0;
        while (iterator.hasNext()) {
            result++;
            iterator.next();
        }
        return result;
    }

    @Override
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    @Override
    public Goods create(Long id) {
        Goods result = null;
        Goods goods = goodsMapper.selectById(id);
        result = productRepository.save(goods);

        return result;
    }

    @Override
    public void batchCreate(List<Long> ids) {
        if (!CollectionUtils.isEmpty(ids)) {
            List<Goods> spxxList = goodsMapper.selectBatchIds(ids);
            productRepository.saveAll(spxxList);
        }
    }

    @Override
    public void batchDelete(List<Long> ids) {
        if (!CollectionUtils.isEmpty(ids)) {
            List<Goods> GoodsList = new ArrayList<>();
            for (Long id : ids) {
                Goods goods = new Goods();
                goods.setId(id);
                GoodsList.add(goods);
            }
            productRepository.deleteAll(GoodsList);
        }
    }

    @Override
    public Page<Goods> search(String keyword, Integer pageNum, Integer pageSize) {
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        return productRepository.findByNameOrKeywords(keyword, keyword, pageable);
    }

    @Override
    public Page<Goods> search(String keyword, Long brandId, Long categoryId, Integer pageNum, Integer pageSize, Integer sort) {
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        NativeSearchQueryBuilder nativeSearchQueryBuilder = new NativeSearchQueryBuilder();
        //分页
        nativeSearchQueryBuilder.withPageable(pageable);
        nativeSearchQueryBuilder.withSort(SortBuilders.scoreSort());
        //过滤
        if (brandId != null || categoryId != null) {
            BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
            if (brandId != null) {
                boolQueryBuilder.must(QueryBuilders.termQuery("brandId", brandId));
            }
            if (categoryId != null) {
                boolQueryBuilder.must(QueryBuilders.termQuery("categoryId", categoryId));
            }
            nativeSearchQueryBuilder.withFilter(boolQueryBuilder);
        }
        //搜索
        if (StringUtils.isEmpty(keyword)) {
            nativeSearchQueryBuilder.withQuery(QueryBuilders.matchAllQuery());
        } else {

            List<FunctionScoreQueryBuilder.FilterFunctionBuilder> filterFunctionBuilders = new ArrayList<>();
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("name", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(10)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("subTitle", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(5)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("keywords", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(2)));
            FunctionScoreQueryBuilder.FilterFunctionBuilder[] builders = new FunctionScoreQueryBuilder.FilterFunctionBuilder[filterFunctionBuilders.size()];
            filterFunctionBuilders.toArray(builders);
            MultiMatchQueryBuilder matchQuery = QueryBuilders.multiMatchQuery(keyword, "name", "subTitle", "keywords");
            FunctionScoreQueryBuilder functionScoreQueryBuilder = QueryBuilders.functionScoreQuery(matchQuery,builders)
                    .scoreMode(FunctionScoreQuery.ScoreMode.SUM)
                    .setMinScore(2);
            nativeSearchQueryBuilder.withQuery(functionScoreQueryBuilder);
        }
        //排序
        if(sort==1){
            //按新品从新到旧
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("id").order(SortOrder.DESC));
        }else if(sort==2){
            //按销量从高到低
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("sale").order(SortOrder.DESC));
        }else if(sort==3){
            //按价格从低到高
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("price").order(SortOrder.ASC));
        }else if(sort==4){
            //按价格从高到低
            nativeSearchQueryBuilder.withSort(SortBuilders.fieldSort("price").order(SortOrder.DESC));
        }else{
            //按相关度
            nativeSearchQueryBuilder.withSort(SortBuilders.scoreSort().order(SortOrder.DESC));
        }
        nativeSearchQueryBuilder.withSort(SortBuilders.scoreSort().order(SortOrder.DESC));
        NativeSearchQuery searchQuery = nativeSearchQueryBuilder.build();
        log.info("DSL:{}", searchQuery.getQuery().toString());
        SearchHits<Goods> searchHits = elasticsearchRestTemplate.search(searchQuery, Goods.class);
        if(searchHits.getTotalHits()<=0){
            return new PageImpl<>(new ArrayList<>(),pageable,0);
        }
        List<Goods> searchProductList = searchHits.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return new PageImpl<>(searchProductList,pageable,searchHits.getTotalHits());
    }


    @Override
    public Page<Goods> recommend(Long id, Integer pageNum, Integer pageSize) {
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        Goods goods = goodsMapper.selectById(id);
        if (Objects.nonNull(goods)) {
            String keyword = goods.getName();
            Long brandId = goods.getBrandId();
            Long categoryId = goods.getCategoryId();
            //根据商品标题、品牌、分类进行搜索
            List<FunctionScoreQueryBuilder.FilterFunctionBuilder> filterFunctionBuilders = new ArrayList<>();
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("name", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(8)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("subTitle", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(2)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("keywords", keyword),
                    ScoreFunctionBuilders.weightFactorFunction(2)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("brandId", brandId),
                    ScoreFunctionBuilders.weightFactorFunction(5)));
            filterFunctionBuilders.add(new FunctionScoreQueryBuilder.FilterFunctionBuilder(QueryBuilders.matchQuery("productCategoryId", categoryId),
                    ScoreFunctionBuilders.weightFactorFunction(3)));
            FunctionScoreQueryBuilder.FilterFunctionBuilder[] builders = new FunctionScoreQueryBuilder.FilterFunctionBuilder[filterFunctionBuilders.size()];
            filterFunctionBuilders.toArray(builders);
            //设置查询条件
            QueryBuilder queryBuilder = QueryBuilders.boolQuery()
                    .must(QueryBuilders.multiMatchQuery(keyword,"name","subTitle","keywords"));
//                    .should(QueryBuilders.matchQuery("brandId", brandId))
//                    .should(QueryBuilders.matchQuery("categoryId", categoryId));
            FunctionScoreQueryBuilder functionScoreQueryBuilder = QueryBuilders.functionScoreQuery(queryBuilder, builders)
                    .scoreMode(FunctionScoreQuery.ScoreMode.SUM)
                    .setMinScore(2);
            //用于过滤掉相同的商品
            BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();
            boolQueryBuilder.mustNot(QueryBuilders.termQuery("id",id));
            //构建查询条件
            NativeSearchQueryBuilder builder = new NativeSearchQueryBuilder();

            builder.withQuery(functionScoreQueryBuilder);
            builder.withFilter(boolQueryBuilder);
            builder.withPageable(pageable);
            NativeSearchQuery searchQuery = builder.build();
            log.info("DSL:{}", searchQuery.getQuery().toString());
            SearchHits<Goods> searchHits = elasticsearchRestTemplate.search(searchQuery, Goods.class);
            if(searchHits.getTotalHits()<=0){
                return new PageImpl<>(new ArrayList<>(),pageable,0);
            }
            List<Goods> searchProductList = searchHits.stream().map(SearchHit::getContent).collect(Collectors.toList());
            return new PageImpl<>(searchProductList,pageable,searchHits.getTotalHits());
        }
        return new PageImpl<>(new ArrayList<>());
    }
}

```



### 6. 事件通知类

添加商品事件

```java
package cn.couldme.core.event;

import lombok.Getter;
import org.springframework.context.ApplicationEvent;

import java.util.List;

/**
 * @Author: szwei
 * @Date: 2020-12-06 23:03
 **/
@Getter
public class GoodsAddEvent extends ApplicationEvent {

    private List<Long> ids;

    public GoodsAddEvent(Object source, List<Long> ids) {
        super(source);
        this.ids = ids;
    }
}

```

删除商品事件

```java
package cn.couldme.core.event;

import lombok.Getter;
import org.springframework.context.ApplicationEvent;

import java.util.List;

/**
 * @Author: szwei
 * @Date: 2020-12-06 23:03
 **/
@Getter
public class GoodsDelEvent extends ApplicationEvent {

    private List<Long> ids;

    public GoodsDelEvent(Object source, List<Long> ids) {
        super(source);
        this.ids = ids;
    }
}

```

### 7. 监听器 

```java
package cn.couldme.es.eventListener;

import cn.couldme.core.event.GoodsAddEvent;
import cn.couldme.core.event.GoodsDelEvent;
import cn.couldme.es.service.EsGoodstService;
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.event.EventListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * @Author: szwei
 * @Date: 2020-12-06 23:22
 **/
@Slf4j
@Component
@AllArgsConstructor
public class GoodsListener {

    private final EsGoodstService esGoodstService;

    @EventListener
    @Async
    public void goodsAddEvent(GoodsAddEvent goodsAddEvent){
        List<Long> ids = goodsAddEvent.getIds();
        log.debug("往es添加商品: ids => {}",ids);
        esGoodstService.batchCreate(ids);
    }

    @EventListener
    @Async
    public void goodsDelEvent(GoodsDelEvent goodsDelEvent){
        List<Long> ids = goodsDelEvent.getIds();
        log.debug("往es删除商品: ids => {}",ids);
        esGoodstService.batchDelete(ids);
    }
}

```

### 8. 配置异步

在启动类添加 `@EnableAsync`注解

### 9. 效果图

![image-20201211153156005](https://gitee.com/szwei/images/raw/master/img/image-20201211153156005.png)

## 五、注意事项

### 1. 启动项目报错

![img](https://gitee.com/szwei/images/raw/master/img/20200827113626915.png)

问题出现原因：

elasticsearch和Redis都需要Netty作为NIO框架，在Redis初始化时已经对Netty进行了初始化处理器数量，当ES再次尝试初始化Netty处理器数量时，Netty就会对此进行保护措施，抛出异常

解决方案：

在启动类上添加：

` System.setProperty("es.set.netty.runtime.available.processors", "false");`

不让es 的 netty 再次去设置



### 2. 查询出来的数据没有按照匹配度排序

在导入数据到 es 的时候需要指定 该字段的使用的分词器

例如：

关键词（不会进行分词）`    @Field(type=FieldType.Keyword)`

最大粒度分词`@Field(analyzer = "ik_max_word",type = FieldType.Text)`





### 

  