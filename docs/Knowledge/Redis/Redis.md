## 一、命令行操作

```shell
## 最后清一下redis缓存
apk add redis
yum install redis

## 连接redis
redis-cli -h 172.20.49.166 -p 6382 -a YVpAXDTN5t

## 选择库
172.20.49.166:6382>select 9

## 查询key
172.20.49.166:6382[9]>keys *

## 删除key
172.20.49.166:6382[9]>del [key...]

## 使用管道删除keys
172.20.49.166:6382[9]>del | keys *123*

## 清空
172.20.49.166:6382[9]>flushdb

## 强制退出redis
172.20.49.166:6382[9]>shutdown
```

