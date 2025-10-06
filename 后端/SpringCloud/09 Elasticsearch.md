
# 基础

### 概述

**Lucene**
- Lucene 是一个 Java语言的搜索引擎类库, 是 Apache 公司的顶级项目
- 官网地址: https://lucene.apache.org
- 优势
	- 易拓展
	- 高性能 (基于倒排索引)

**Elasticsearch**
- 2004年, Shay Banon 基于 Lucene 开发了 Compass
- 2010年, Shay Banon 重写了 Compass, 取名为 Elasticsearch
- 官网地址: https://www.elastic.co/cn/
- 优势
	- 支持分布式, 可水平拓展
	- 提供 Restful 接口, 可以被任何语言调用
- ES 结合 kibana, Logstash, Beats, 是一套技术栈, 被称为 ELK
- 该套技术栈被广泛应用在日志数据分析, 实时监控等领域

**安装与配置**
- 安装 elasticsearch
- 分配给虚拟机更大的内存后可以给es分配更大的内存, 会获得更好的体验
- 使用 9200 端口访问 es
```bash
docker run -d \
  --name es \
  -e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
  -e "discovery.type=single-node" \
  -v es-data:/usr/share/elasticsearch/data \
  -v es-plugins:/usr/share/elasticsearch/plugins \
  --privileged \
  --network hm-net \
  -p 9200:9200 \
  -p 9300:9300 \
  elasticsearch:7.12.1
```
- 安装 kibana
- 使用5601访问控制台
- 使用 dev_tools 可以向 es 中发送 http 请求, 在其中, 只需要 `/` 即可访问es服务器的根目录, 同时 dev_tools 中存在提示, 方便访问
```shell
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=hm-net \
-p 5601:5601  \
kibana:7.12.1
```

### 倒排索引

- 传统数据库 (如 MySQL) 使用正向索引, 不擅长文本中对词语的模糊搜索
	- 搜索时, 先找到文档, 再搜索其中是否包含词条
- `elasticsearch` 采用倒排索引, 分为两个概念
	- 文档 (document) : 每条数据就是一个文档
	- 词条 (term) : 文档按照**语义**分成的词语
	- 建立一个表, 按照词条存储文档 id
- 用户搜索时, 先将用户的输入划分为词条, 再按词条进行文档搜索, 每个词条都相当于存在索引, 所以搜索速度快

### IK分词器

**概述**
- 中文分词往往需要根据语义分析, 比较复杂, 这就需要用到**中文分词器**
- **IK分词器** 是林益良在2006年开源发布的, 其使用的正向迭代最细粒度切分算法一直沿用至今
- 安装时, 只需要将其分词器加入 `elasticsearch` 的插件目录即可

**使用**
- 在 `kibana` 中使用如下语法测试 ik 分词器
- `POST`: 请求方式
- `/_analyze`: 请求路径, 此处省略了es完整路径, 由 `kibana` 帮助补充
- 请求参数: JSON 风格
	- `analyzer`: 分词器类型
		- `standard` : 默认分词器, 中文分词效果不理想
		- `ik_smart` : ik分词器, 按照中文语义进行分词
		- `ik_max_word` : ik分词器, 会将原有的词语保留, 并进一步细分
	- `text`: 要分词的内容
```
POST /_analyze
{
  "analyzer": "standard",
  "text": "黑马程序员学习Java太棒了"
}
```

**拓展**
- 由于网络词汇不断产生变化, 新词汇不断产生, 所以需要有能够拓展原有词典的能力, ik 分词器允许配置自定义词典来扩充词库
	- 停止词: 语句中没有实际含义的词汇, 如语气词
- 扩充的词典与配置文件的需要在同一个目录下
- 注意: 配置文件需要是 不带 BOM 头, 以 LF 换行的 UTF-8 文件, 不能出现错误

### 概念
- `elasticsearch` 中的文档数据会被序列化为 json 格式后存储在 es 中
- 索引 (index): 相同类型的文档的集合
- 映射 (mapping): 索引中文档的字段约束信息, 类似表的约束

**与MySQL对比**
- `Table` -> `Index` : 索引, 就是文档的集合, 类似数据库的表
- `Row` -> `Document` : 文档, 就是一条条数据, 类似数据库中的行, 以 `JSON` 存储
- `Column` -> `Field` : 字段, 就是 `JSON` 文档中的字段, 类似数据库中的列
- `Schema` -> `Mapping` : 映射是索引中对文档的约束, 类似数据库中的表结构
- `SQL` -> `DSL` : `DSL` 是 `ES` 提供的 `JSON` 风格的请求语句, 用来定义搜索条件

# 基本操作

### Mapping映射属性

- mapping 是对索引库中文档的约束, 常见的mapping属性包括
- type: 字段数据类型, 常见简单类型有:
	- 字符串: text (可分词文本), keyword (精确值, 如国家, 城市名等)
	- 数值: long, integer, short, byte, double, float
	- 布尔: boolean
	- 日期: date
	- 对象: object
- index: 是否创建索引, 默认为 true
- analyzer: 使用哪种分词器
- properties: 该字段的子字段

### 索引库操作

**概述**
- Elasticsearch 提供的所有 API 都是 Restful 的接口, 遵循 Restful 的基本规范

**获取索引库**
```json
get /heima
```

**删除索引库**
```json
delete /heima
```

**创建索引库和mapping**
- 示例
```json
PUT /heima
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "age": {
        "type": "byte"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object", 
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

**修改索引库**
- 索引库和mapping一旦创建无法修改, 但是可以添加新的字段
```json
PUT /heima/_mapping
{
  "properties": {
    "age": {
      "type": "byte"
    }
  }
}
```

### 文档CRUD

- 新增文档的请求格式如下
- 返回 201, 表示新增成功
```json
POST /heima/_doc/1
{
  "info": "黑马程序员Java讲师",
  "email": "zy@163.com",
  "age": 18,
  "name": {
    "firstName": "云",
    "lastName": "赵"
  }
}
```
- 修改文档
- 方法1: 全量修改, 删除旧文档, 添加新文档
- PUT 请求方法同样可以用来新增文档
```json
PUT /heima/_doc/1
{
  "info": "黑马程序员Java讲师",
  "email": "zy@163.com",
  "age": 99,
  "name": {
    "firstName": "云",
    "lastName": "赵"
  }
}
```
- 方法2: 增量修改, 只修改指定字段
```json
POST /heima/_update/1
{
  "doc": {
    "字段名": "新值"
  }
}
```

- 查询文档
```json
GET /heima/_doc/1
```
- 删除文档
```json
DELETE /heima/_doc/1
```

### 文档批量处理

- ES 允许一次请求中携带多次文档操作, 也就是批量处理
- 操作 `_bulk` 路径并提供多行 JSON 实现批处理操作
- 语法如下 - 删除操作只需要一行, 其余都需要两行, 且不能换行
```json
POST /_bulk
{"index": {"_index": "heima", "_id": "3"}}
{"info": "Python讲师", "email": "123@163.com"}
{"update": {"_id": "1", "_index": "hmall"}}
{"doc": {"email": "123@163.com"}}
{"delete": {"_index": "heima", "_id": "3"}}
```

# JavaRestClient基础

- Elasticsearch 目前的最新版本已经到8.0+, 其 JavaClient 存在很大变化
- 由于多数企业使用的是 8以下 版本, 所以选择早期的 JavaRestClient
- 文档地址: 

### 初始化

**客户端初始化**
- 引入依赖
```xml
<!-- ES客户端依赖 -->  
<dependency>  
    <groupId>org.elasticsearch.client</groupId>  
    <artifactId>elasticsearch-rest-high-level-client</artifactId>  
</dependency>
```
- 由于 SpringBoot 的默认 ES 版本与使用的不符, 所以需要在配置中进行覆盖
- 覆盖时应在父工程进行, 指定依赖版本
```xml
<properties>  
    <elasticsearch.version>7.12.1</elasticsearch.version>  
</properties>
```
- 初始化 RestHighLevelClient
- 非容器管理, 手动创建和销毁
```java
private RestHighLevelClient client;
@BeforeEach
void setUp() {  
    client = new RestHighLevelClient(RestClient.builder(  
            HttpHost.create("192.168.199.200:9200")  
    ));  
}
@AfterEach  
void tearDown() throws IOException {  
    if(client != null) {  
        client.close();  
    }  
}
```

### 索引库操作

**商品Mapping映射**
- 设置索引库时, 要结合搜索需求设计索引库的字段
- 综合考虑搜索所需字段, 商品排序方式, 搜索结果所需字段
- **注意**: 设计索引库时, 通常将 id 的 `type` 设置为 `keyword` 类型而不是整型

**JavaAPI**
- 创建索引库请求 `Request` 对象
- 设置请求参数, 传入模板和请求格式
- 发起请求, 完成索引库创建
- `client.indices()` 中包含了对索引库的所有操作方法

**创建索引库**
- `MAPPING_TEMPLATE` 为 `JSON` 数据, 保存在字符串常量中
- 注意: 如果索引库已存在, 则不会被覆盖, 需要提前删除索引库
```java
@Test  
void testCreateIndex() throws IOException {  
    // 准备Request对象  
    CreateIndexRequest request = new CreateIndexRequest("items");  
    // 准备请求参数  
    request.source(MAPPING_TEMPLATE, XContentType.JSON);  
    // 发送请求  
    client.indices().create(request, RequestOptions.DEFAULT);  
}
```
**查询索引库**
```java
@Test  
void testGetIndex() throws IOException {  
    GetIndexRequest request = new GetIndexRequest("items");  
    client.indices().get(request, RequestOptions.DEFAULT);  
}
```
**删除索引库**
```java
@Test  
void testDeleteIndex() throws IOException {  
    DeleteRequest request = new DeleteRequest("items");  
    client.delete(request, RequestOptions.DEFAULT);  
}
```

### 文档操作

- 与索引库操作类似
- 首先创建 `Request` 请求对象
- 然后使用 `JSON` 初始化请求对象
- 使用 `client.index()` 发送文档操作请求

**新增文档**
- 创建一个与 Mapping 中结构一致的类
```java
@Data  
@ApiModel(description = "索引库实体")  
public class ItemDoc{  
    @ApiModelProperty("商品id")  
    private String id;  
    ...
}
```
- 进行查询
```java
@Test  
void testIndexDoc() throws IOException {  
    // 准备文档数据  
    Item item = itemService.getById(100000011127L);  
    ItemDoc itemDoc = BeanUtil.copyProperties(item, ItemDoc.class);  
    String itemDocJson = JSONUtil.toJsonStr(itemDoc);  
    // 准备 Request    
    IndexRequest request = 
        new IndexRequest("items").id(itemDoc.getId());  
    // 准备请求参数  
    request.source(itemDocJson, XContentType.JSON);  
    // 发送请求  
    client.index(request, RequestOptions.DEFAULT);  
}
```

**删除文档**
```java
@Test  
void testDeleteDoc() throws IOException {  
    DeleteRequest request = new DeleteRequest("items", "100000011127");  
    client.delete(request, RequestOptions.DEFAULT);  
}
```
**查询文档**
```java
@Test  
void testGetDoc() throws IOException {  
    GetRequest request = new GetRequest("items", "100000011127");  
    GetResponse response = client.get(request, RequestOptions.DEFAULT);  
    ItemDoc itemDoc = JSONUtil.toBean(response.getSourceAsString(), ItemDoc.class);  
    System.out.println(itemDoc);  
}
```

**修改文档**
- 全量更新: 再次写入相同 id 文档, 会删除旧文档, 与新增的 JavaAPI 完全一致
- 增量更新: 只更新部分指定字段, 每两个参数为一个 key-value
```java
@Test  
void testUpdateDoc() throws IOException {  
    UpdateRequest request = new UpdateRequest("items", "100000011127");  
    request.doc(  
            "price", 25600  
    );  
    client.update(request, RequestOptions.DEFAULT);  
}
```

### 文档批处理

- 批处理代码流程与之前类似, 只是使用 `BulkRequest` 封装普通的 `CRUD` 请求
- 使用命令 `GET /item/_count` 可查看表中数据总量
- 代码示例
```java
@Test  
void testBulkTest() throws IOException {  
    // 准备数据  
    int pageNo = 1, pageSize = 500;  
    // 翻页, 循环写入  
    while (true) {  
        BulkRequest request = new BulkRequest();  
        Page<Item> page = itemService.lambdaQuery()  
                .eq(Item::getStatus, 1)  
                .page(Page.of(pageNo, pageSize));  
        List<Item> records = page.getRecords();  
        if (records == null || records.isEmpty()) {  
            return;  
        }  
        // 写入批量处理 Request       
        for (Item record : records) {  
            ItemDoc itemDoc = 
                BeanUtil.copyProperties(record, ItemDoc.class);  
            request.add(  
                    new IndexRequest("item").id(itemDoc.getId())  
                    .source(JSONUtil.toJsonStr(itemDoc))  
            );
        }
        // 发送请求
        client.bulk(request, RequestOptions.DEFAULT);  
        // 翻页
        pageNo++;
    }
}
```

# DSL查询

### 概述

**概念**
- `Elasticsearch` 提供了 `DSL` 查询, 就是以 `JSON` 格式定义查询条件
- `DSL` 查询可以分为两大类
	- **叶子查询**: 一般在特定字段中查询特定值, 属于简单查询, 很少单独使用
	- **复合查询**: 以逻辑方式组合多个叶子查询或更改叶子查询的行为方式
- 在查询后, 还可以对结果做处理, 包括
	- 排序: 按1个或多个字段值进行排序
	- 分页: 根据 from 和 size 做分页, 类似 MySQL
	- 高亮: 对搜索结果中的关键字添加特殊样式, 使其更加醒目
	- 聚合: 对搜索结果做数据统计以形成报表

**基础使用**
- 对于查询全部方法 `match_all`
- ES 内部默认单次查询最大允许数据为10000条
- ES 内部默认最多返回10条数据
- 基于 DSL 的查询语法如下
```json
GET /indexName/_search
{
  "query": {
    "查询类型": {
      "查询条件": "条件值"
    }
  }
}
```

### 叶子查询
- 叶子查询还可以进一步细分, 常见的有以下

##### 全文检索
- 利用分词器对用户输入内容分词, 然后去词条列表中匹配
- `match` 查询: 全文检索查询的一种, 会对用户输入内容分词, 然后去倒排索引库进行检索
- `field`: 查询条件; `text`: 查询内容
```json
GET /indexName/_search
{
  "query": {
    "match": {
      "field": "text"
    }
  }
}
```
- `multi_match`: 与 `match` 查询类似, 但是允许同时查询多个字段
- `query`: 查询条件; `fields`: 欲查询的字段组合
```json
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "text",
      "fields": ["field1", "field2", ...]
    }
  }
}
```

##### 精确查询
- 不对用户输入内容分词, 直接精确匹配, 通常用来查找 keyword, 数值, 日期等
- 例如: 
	- ids: 根据 id 查询
	- range: 根据范围查询
	- term: 词条查询

**term**
- 可以查询词条中的所有对应项
```json
GET /indexName/_search
{
  "query":{
    "term": {
      "field": {
        "value": "VALUE"
      }
    }
  }
}
```
**range**
- `gt`: 大于; `gte`: 大于等于
```json
GET /indexName/_search
{
  "query":{
    "range": {
      "field": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```
**ids**
```json
GET /indexName/_search
{
  "query":{
    "ids": {
      "values": ["id1", "id2", ...]
    }
  }
}
```

**地理(geo)查询**
- 用于搜索地理位置, 搜索方式很多, 例如
	- geo_distance
	- geo_bounding_box

### 复合查询

- 复合查询大致可分为两类
- 第一类: 基于逻辑运算组合叶子查询, 实现组合条件
	- bool
- 第二类: 基于某种算法修改查询时的文档相关性算分, 从而改变文档排名
	- function_score
	- dis_max

**布尔查询**
- 布尔查询是一个或多个查询子句的组合.
- 选择组合方式时, 对于不必要算分的应当不参与算分, 以提高搜索效率
- 子查询组合方式如下
- `must`: 必须匹配每个子查询, 类似 与
- `should`: 选择性匹配子查询, 类似 或
- `must_not`: 必须不匹配, 不参与算分, 类似 非
- `filter`: 必须匹配, 不参与算分

**案例**
- 搜索智能手机, 品牌为华为, 价格介于900~1599
```json
GET /items/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "智能手机"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "brand": "华为"
          }
        },{
          "range": {
            "price": {
              "gte": 90000,
              "lte": 159900
            }
          }
        }
      ]
    }
  }
}
```

### 排序与分页

##### 排序
- ES 支持对搜索结果进行排序, 默认是根据相关度算分 (`_score`) 来排序, 也可以指定字段排序, `asc` 为升序, `desc` 为降序
- 拥有多个排序字段, 先按第一个进行排序, 如果相同再按第二个, 依次类推
- 可以排序的字段类型有: keyword类型, 数值类型, 地理坐标类型, 日期类型等
```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "field": "desc"
    }
  ]
}
```

**案例**
- 对搜索商品按销量排序, 销量一致时按照价格进行排序
- 排序完整写法为: `"sort": [{"FIELD": {"order": "desc"}}]`
- 排序简化写法为: `"sort": [{"FIELD": "desc"}]`
```json
GET /items/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 10, 
  "sort": [
      {
        "sold": "desc"
      },
      {
        "price": "asc"
      }
  ]
}
```

##### 分页
- `elasticsearch` 默认情况下只返回前10条数据, 如果需要查询更多的数据就需要修改分页参数.
- `ES` 中通过修改 `from` 和 `size` 参数来控制要返回的分页结果
- from: 从第几页开始; size: 需要查询几个文档
- 案例: 搜索商品, 查询销量前10的商品, 如果销量相同按价格升序
```json
GET /items/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 10, 
  "sort": [
      {
        "sold": "desc"
      },
      {
        "price": "asc"
      }
  ]
}
```

**深度分页问题**
- elasticsearch 中的数据一般会进行**分片存储**, 也就是把一个索引中的数据分为 N 份, 存储到不同节点上, 查询数据时需要汇总各个分片的数据
- 由于分片存储的存在, 在进行分页查询时, 要先在每个分片中取出其进行排序后的前n个, 再将每页前n个合起来一起排序后取前n个, 耗时较多
- 分页越深, 需要查询的数据量越大, 导致查询速度过慢甚至直接榨干内存

**解决**
- 大多数公司对深度分页的解决方案都是直接设置**分页上限**
- ES 官方对于分页同样设置了上限, 即 `from + size <= 10000`
- 针对深度分页问题, 官方提供了两种解决方案
- search after: 分页时进行排序, 原理是从上一次排序值开始, 查询下一页数据, 官方推荐使用的格式
	- 优点: 没有查询上限, 支持深度分页
	- 缺点: 只能向后逐页查询, 不能随机翻页
	- 场景: 数据迁移, 手机滚动查询
- scroll: 原理是将排序数据形成快照保存在内存中. 官方已不推荐使用

### 高亮显示

- 高亮显示就是把关键字在搜索结果中突出显示
- 倒排索引在记录时, 不仅记录文档 id, 还会记录相关索引项在文档中的位置
- 只需要在查询时指定, 就可以在指定文档内容索引项前后加入相关标签
- 不知道具体标签时, 默认添加前后标签即为 `<em></em>`
```json
GET /items/_search
{
  "query": {
    "match": {
      "name": "脱脂牛奶"
    }
  },
  "highlight": {
    "fields": {
      "name": {
        "pre_tags": "<em>",
        "post_tags": "</em>"
      }
    }
  }
}
```


# JavaRestClient查询

### 概述

- 数据搜索的 Java 代码可以分为两部分
1. 构建并发起请求
2. 解析查询结果
```java
@Test  
void testMatchAll() throws IOException {  
    // 构建和发起请求
    SearchRequest request = new SearchRequest("items");  
    request.source().query(QueryBuilders.matchAllQuery());  
    SearchResponse response = 
        client.search(request, RequestOptions.DEFAULT);  
    // 解析查询结果
    SearchHits responseHits = response.getHits();  
    long count = responseHits.getTotalHits().value;  
    System.out.println("count = " + count);  
    SearchHit[] resultHits = responseHits.getHits();  
    for (SearchHit hit : resultHits) {  
        ItemDoc itemDoc = 
            JSONUtil.toBean(hit.getSourceAsString(), ItemDoc.class);  
        System.out.println(itemDoc);  
    }  
}
```

### 查询条件

**概述**
- 在 JavaRestAPI 中, 所有类型的 query 查询条件都是由 QueryBuilders 构建的

**全文检索**
- 单字段查询: `matchQuery(FIELD, VALUE)`
- 多字段查询: `multiMatchQuery(VALUE, FIELD1, FIELD2, ...)`

**精确查询**
- 词条查询: `termQuery(FIELD, VALUE)`
- 范围查询: `rangeQuery(FIELD).gte(VALUE1).lte(VALUE2)`

**布尔查询**
- 创建布尔查询: `QueryBuilders.boolQuery()`
- 添加 must 条件: `boolQuery.must(QueryBuilders.termQuery(xxx))`
- 添加 filter 条件: `boolQuery.filter(xxx)`
- 案例: 搜索关键字为牛奶, 品牌为德亚, 价格低于300的所有商品
```java
@Test  
void testBoolSearch() throws IOException {  
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();  
    boolQuery.must(  
            QueryBuilders.matchQuery("name", "脱脂牛奶")  
    ).filter(  
            QueryBuilders.termQuery("brand", "德亚")  
    ).filter(  
            QueryBuilders.rangeQuery("price").lt(30000)  
    );  
    SearchRequest request = new SearchRequest("items");  
    request.source().query(boolQuery);  
    ...  
}
```

### 排序和分页

- 与 `query` 类似, 排序和分页都是基于 `request.source()` 进行设置
```java
@Test  
void testSortAndPage() throws IOException {  
    SearchRequest request = new SearchRequest("items");  
    request.source().query(QueryBuilders.matchAllQuery());  
    // 分页  
    request.source().from(0).size(5);  
    // 排序, 按销量降序  
    request.source().sort("sold", SortOrder.DESC)  
            .sort("price", SortOrder.ASC);  
    SearchResponse response = 
        client.search(request, RequestOptions.DEFAULT);  
    showResponse(response);  
}
```

### 高亮显示
- 同样基于 `source()` 进行设定, 但是其参数是基于 `SearchSourceBuilder` 中的 `highlight()` 方法进行指定的, 不能直接设置
- 高亮的显示结果不存储在 `source` 字段, 而是存储在 `highlight` 字段中, 必须将 `highlight` 中的值取出后替换掉原本 `source` 字段的值
- `ES` 在查找高亮片段时, 会对原有字符串内容进行切割, 查找完毕后, 会将这些切片放入一个数组中, 使用时, 应将所有字符串进行组装成为新高亮字符串
```java
@Test  
void testHighLight() throws IOException {  
    SearchRequest request = new SearchRequest("items");  
    request.source().query(QueryBuilders.matchQuery("name", "脱脂牛奶"));  
    // 高亮  
    request.source().highlighter(  
            SearchSourceBuilder.highlight()  
                    .field("name")  
    );  
    SearchResponse response = 
        client.search(request, RequestOptions.DEFAULT);  
    long count = response.getHits().getTotalHits().value;  
    SearchHit[] hits = response.getHits().getHits();  
    for (SearchHit hit : hits) {  
        String source = hit.getSourceAsString();  
        ItemDoc itemDoc = JSONUtil.toBean(source, ItemDoc.class);  
        // 获取高亮结果  
        Map<String, HighlightField> hlf = hit.getHighlightFields();  
        if(CollUtil.isNotEmpty(hlf)) {  
            HighlightField field = hlf.get("name");  
            if(field != null) {  
                String highlightName = 
                    field.getFragments()[0].toString();  
                itemDoc.setName(highlightName);  
            }  
        }  
        System.out.println(itemDoc);  
    }  
}
```

# 数据聚合

**概述**
- 聚合 (aggregation) 可以实现对文档数据的统计, 分析, 运算.
- 参与聚合的字段必须是 `Keyword`, 数值, 日期, 布尔 等不可被分词的字段

聚合常见的有三类
- 桶 (Bucket) 聚合: 用来对文档做分组
	- Term Aggregattion: 按照文档字段值分组
	- Date Histogram: 按照日期阶梯分组, 如一周为一组或者一月为一组
- 度量 (Metric) 聚合: 用以计算一些值, 如最大值, 最小值, 平均值等
	- Avg: 平均值
	- Max: 最大值
	- Min: 最小值
	- Stats: 同时求 max, min, avg, sum 等
- 管道 (pipeline) 聚合: 以其他聚合的结果为基础做聚合

### DSL聚合
- 当搜索所有时, 直接省略 `query` 即可
- 设置 size 为 0 时, 结果中将不包含文档数据, 只包含聚合结果
- 聚合三要素: 聚合名称, 聚合类型, 要聚合的字段, 希望获取的聚合结果数量
- 默认情况下, Bucket 聚合是对所有文档的聚合, 需要限定文档范围时, 只需要添加 query 条件即可

**案例** : 统计所有商品中共有哪些商品分类, 即以 category 进行分组
```json
GET /items/_search
{
  "size": 0,
  "aggs": {
    "category_agg": {
      "terms": {
        "field": "category",
        "size": 5
      }
    }
  }
}
```
**案例** : 统计价格高于3000的所有手机的商品分类
```json
GET /items/_search
{
  "size": 0, 
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "category": "手机"
          }
        },
        {
          "range": {
            "price": {
              "gte": 300000
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "cate_agg": {
      "terms": {
        "field": "brand",
        "size": 10
      }
    }
  }
}
```
- **案例**: 在分组基础上, 对每个分组数据做计算和统计
```json
GET /items/_search
{
  "size": 0, 
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "category": "手机"
          }
        }
      ]
    }
  },
  "aggs": {
    "cate_agg": {
      "terms": {
        "field": "brand",
        "size": 10
      },
      "aggs": {
        "price_stats": {
          "stats": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

### JavaClient聚合

- 在 `source()` 的基础上调用 `aggregation()` 即可设置聚合参数
- 在函数中传入参数 `AggregationBuilders` 设置不同聚合类型的聚合方法
- `Aggregation` 是一个顶级接口, 需要自己选定聚合类型
```java
@Test  
void testAgg() throws IOException {  
    SearchRequest request = new SearchRequest("items");  
    // 查询数量设为0  
    request.source().size(0);  
    // 聚合条件  
    String brandAggName = "brandAgg";  
    request.source().aggregation(  
            AggregationBuilders  
                    .terms(brandAggName)  
                    .field("brand")  
                    .size(5)  
    );  
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);  
    // 根据聚合名称获取聚合  
    // Aggregation是一个顶级接口, 需要自己选定聚合类型  
    Terms terms = response.getAggregations().get(brandAggName);  
    List<? extends Terms.Bucket> buckets = terms.getBuckets();  
    for (Terms.Bucket bucket : buckets) {  
        System.out.println("brand: " + bucket.getKey());  
        System.out.println("count: " + bucket.getDocCount());  
    }  
}
```
