
# 关系方向
```
from ------> to
从from到to,是出去,所以是OUTBOUND

from <------to  
从to到from,是回来,所以是INBOUND
```

# 查某个人的外向关系
```
WITH CustomIds,HomeAddr,WorkNames,HouseAddr 
FOR v, e, p IN 1..2 ANY 'CustomIds/3D0B57F2C682A67E42EF8567C43ADF0C' CustomIds_HomeAddr,customeid_workName,CustomIds_HouseAddr
 return p
```
遍历这个人,找到这个人的1度内向关系
```
FOR c IN Characters
    FILTER c.name == "Ned"
    FOR v IN 1..1 INBOUND c ChildOf
        RETURN v.name
```
注意:如果是集群模式,那么需要写上with语句.如果是单机模式,那么写不写with语句都行

