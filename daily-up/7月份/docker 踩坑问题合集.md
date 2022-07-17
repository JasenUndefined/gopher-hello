docker 踩坑问题合集

> 未发布

1、使用docker-compose 创建 容器时，报错

Unable to retrieve version information from Elasticsearch nodes. socket hang up

2、报错：# FATAL Error: EISDIR: illegal operation on a directory, read #608

配置的路径没有正确读取到配置文件



查看 elasticsearch 的IP

docker inspect elasticsearch | grep IPAddress



使用root 权限进入 docker 的容器内

 docker exec -u 0 -it 004e8d065b43 /bin/sh
