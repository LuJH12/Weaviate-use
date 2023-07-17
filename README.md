# Weaviate-use
由于官网的教程写得比较复杂，所以笔者写一个简单的例子

可以看[jupyter](https://github.com/LuJH12/Weaviate-use/blob/main/Weaviate_example.ipynb)文件，里面有详细的注释

# 安装

## Docker
网上教程较多，这里就不赘述了。

## Weaviate安装
这里的安装是使用docker进行安装，所以请务必先安装好docker。

[官网安装](https://weaviate.io/developers/weaviate/installation/docker-compose)方法：
![test image size](https://github.com/LuJH12/Weaviate-use/blob/main/figure/Weaviate_install.png){:height="50%" width="50%"}
打开官网后，会看到这个界面，自己选择需要安装的版本、模块等。在选择完成后，可以在下面看到给你生成的一个串命令。

我这里的安装是选择了最简单的(全默认)，生成了下列命令，并在命令行中输入
```
curl -o docker-compose.yml "https://configuration.weaviate.io/v2/docker-compose/docker-compose.yml?modules=standalone&runtime=docker-compose&weaviate_version=v1.20.1"
```
在下载该文件后，在命令行中输入以下命令
```
docker-compose up -d
```
等待安装完成后，可运行下述命令检查是否安装完成
```
docker ps -a
```
如果成功安装，是有weaviate的镜像会显示的
