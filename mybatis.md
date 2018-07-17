###mybatis 

* 使用mybatis  如果出现在数据库查询出多条，但在mybatis中返回一条，那么可以有两种解决方法：
  * 判断你所关联表的一对多关系，合理使用<association> 与 <collection> 标签  这样的话 代码中只会返回一条记录，但是会有一个list在他其中
  * 把你关联表的主键 全部显式的指定出来  
  ```
   <id column="place_id" property="place.placeId"/>
   <id column="check_item_id" property="checkItems.checkItemId"/>
   <id column="check_result_id" property="checkResults.checkResultId"/>
   
   <association property="user" resultMap="UserMap"/>
   <association property="place" resultMap="PlaceMap"/>
   <collection property="checkItems" resultMap="CheckItemMap"/>
