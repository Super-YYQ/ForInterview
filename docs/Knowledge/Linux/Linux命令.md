## 一、日志查询

```shell
cd /data/logs/app

## cat -n 查询出需要的行号
cat -n [log-name] | grep "搜索内容" -C 200

## 查询100~150行的日志
## tail -n +100  表示查询100行之后的数据
## head -n 50 表示往后差50行
cat -n [log-name] | tail -n +100 | head -n 50
```

## 二、SSH

```shell
$ ssh -p 2222 root@192.168.22.12
```

## 三、zip命令

```shell
## 安装unzip命令
sudo apt install unzip

## 将file.zip解压到当前目录unzipped_directory文件夹下
unzip file.zip -d unzipped_directory
```

## 四、容器命令

```shell
## 查询所有容器，过滤关键词
kubectl get pod -A | grep mdd-bd-package

## 将文件test放入容器中/usr/local/bin目录下
kubectl -n c87e2267-1001-4c70-bb2a-ab41f3b81aa3 cp test online-bd-package-7cb6ff7f97-sk8bb:/usr/local/bin
```

## 五、查询

```shell
## 查询文件名   路径   文件名*
find  /usr/local/tomcat/webapps/mdd-bd-package/WEB-INF/lib -name  'mdd-ext-common*'
```

