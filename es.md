```java
这是一些es操作的基本命令

//检查健康状况
GET /_cat/health?v
//节点状态
GET /_cat/nodes?v
//索引状态
GET /_cat/indices?v

//插入索引
PUT /customer?pretty
//插入索引/类型/id/文档 John Doe
PUT /customer/external/1?pretty
{
    "name": "John Doe"
}
//更新上边
PUT /customer/external/1?pretty
{
  "name": "lta"
}
//在customer索引external类型下创建一个id为2的文档
PUT /customer/external/2?pretty
{
  "name": "Jane Doe"
}
//让es默认生成一个id的文档
POST /customer/external/?pretty
{
  "name": "Jane Doe"
}
// 更新文档
POST /customer/external/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
//更新文档中的某一个字段age
POST /customer/external/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
//删除id=2的
DELETE /customer/external/2?pretty
//获取id为1的
GET /customer/external/1?pretty
//删除索引
DELETE /customer?pretty

//批量创建
POST /customer/external/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe1212122" }
{"index":{"_id":"2"}}
{"name": "Jane Doe3333333" }

//批量操作
POST /customer/external/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
