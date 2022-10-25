# compose-collect

## Flume

1.构建容器

```
docker-compose build
```

2.启动

```
docker-compose up
```

source中新建test_1.log，向文件中新增内容，将打印到控制台中



直接运行容器启动flume

```
docker run -it --rm --name flume_test -v $PWD/code/localhost/flume/:/var/flume/ compose-collect-flume:latest /bin/bash
```

```
flume-ng agent -n a1 -c /opt/flume/conf -f /var/flume/conf/test.conf -Dflume.root.logger=INFO,console
```

