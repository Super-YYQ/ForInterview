## 一、日志查询

```shell
cd /data/logs/app

## cat -n 查询出需要的行号
cat -n [log-name] | grep "搜索内容"

## 查询100~150行的日志
## tail -n +100  表示查询100行之后的数据
## head -n 50 表示往后差50行
cat -n [log-name] | tail -n +100 | head -n 50
```

