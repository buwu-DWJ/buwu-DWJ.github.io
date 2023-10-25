
|                            |                                                                                                         |
| -------------------------- | ------------------------------------------------------------------------------------------------------- |
| 选择切换数据库：           | use articledb                                                                                           |
| 插入数据：                 | db.comment.insert(bson数据)                                                                             |
| 查询所有数据：             | db.comment.find()；                                                                                     |
| 条件查询数据：             | dlb.comment.find({条件})                                                                                |
| 查询符合条件的第一条记录： | db.comment.findone({条件})                                                                              |
| 查询符合条件的前几条记录： | db.comment.find({条件}).limit(条数)                                                                     |
| 查询符合条件的跳过的记录： | db.comment.find({条件}).skip(条数)                                                                      |
| 修改数据：                 | db.comment.update({条件}，{修改后的数据})或db.comment.update({条件}，{\$set：{要修改部分的字段: 数据}}) |
| 修改数据并自增某字段值：   | db.comment.update({条件}，{\$inc：{自增的字段：步进值}})                                                |
| 删除数据：                 | db.comment.remove({条件})                                                                               |
| 统计查询：                 | db.comment.count({条件})                                                                                |
| 模糊查询：                 | db.comment.find({字段名：/正则表达式/})                                                                 |
| 条件比较运算：             | db.comment.find({字段名：{\$gt：值}})                                                                   |
| 包含查询：                 | db.comment.find({字段名：{\$in：［值1，值2］}})或db.comment.find({字段名：{\$nin：［值1，值2］}})       |
| 条件连接查询：             | db.comment.find({\$and：［{条件1}，{条件2}］})或db.comment.find({\$or：［{条件1}，{条件2}］})           |