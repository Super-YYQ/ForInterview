## 一、管理语句

```shell
# 登录mysql
$ mysql -h[127.0.0.1] -u[用户名] -p[密码] [数据库];
```

![image-20210222165709584](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210222165709584.png)

```shell
# 查询现有数据库
mysql> show datebases;
# 进入某个数据库
mysql> use [数据库名];
```

![image-20210222170142367](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210222170142367.png)

```shell
# 显示表状态
mysql> show table status like "[表名]";
```

![image-20210222170832681](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210222170832681.png)

```shell
# 查询语句执行计划
mysql> explain [select * from user];
```

![image-20210222171028320](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210222171028320.png)

```powershell
# 查询数据库正在执行的事务
mysql> SELECT * FROM information_schema.INNODB_TRX
# 杀掉某事务
mysql> kill [trx_mysql_thread_id]
```

![image-20210223135713743](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210223135713743.png)

```powershell
# 查看默认的存储引擎
mysql> show variables like '%storage_engine%';
```

![image-20210223172301907](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210223172301907.png)

```shell
# 查询mysql提供的所有存储引擎
mysql> show engines;
```

![image-20210223172249148](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210223172249148.png)