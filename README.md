# Weaviate-use
由于官网的教程写得比较复杂，所以笔者写一个简单的例子，注意：本教程只作简单使用(这个例子只是举个例子，并未追求好的检索效果)。

可以看[jupyter](https://github.com/LuJH12/Weaviate-use/blob/main/Weaviate_example.ipynb)文件，里面有详细的注释

# 安装

## Docker
网上教程较多，这里就不赘述了。

## Weaviate安装
这里的安装是使用docker进行安装，所以请务必先安装好docker。

[官网安装](https://weaviate.io/developers/weaviate/installation/docker-compose)方法：

打开官网后，会看到这个界面，自己选择需要安装的版本、模块等。在选择完成后，可以在下面看到给你生成的一个串命令。

![官网的Weaviate安装选择](https://github.com/LuJH12/Weaviate-use/blob/main/figure/Weaviate_install.png)

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

![docker_weaviate](https://github.com/LuJH12/Weaviate-use/blob/main/figure/docker_weaviate.png)

## 安装Weaviate的python库
```
pip install weaviate-client
```

# 如何在Python上使用Weaviate
这里使用了一个自己随便构建的数据集，是一个有20条

先导包
```python
import weaviate
from langchain.document_loaders import DirectoryLoader, WebBaseLoader
import pandas as pd
```

定义下Weaviate的参数等
```python
# 定义client
client = weaviate.Client(url='http://localhost:8080')
class_name = 'Stephen_Chow'  # class的名字

class_obj = {
    'class': class_name,         # class的名字
    'vectorIndexConfig':{
        'distance': 'l2-squared',   # 这里的distance是选择向量检索方式，这里选择的是欧式距离
    },
}
```

创建class
```python
client.schema.create_class(class_obj)
```

数据导入，我这里使用的是自己构建的一个关于周星驰台词的数据，长度为20，格式为csv
```
# 导入数据
df = pd.read_csv('data.csv', encoding='GB18030')
# 转成list形式
sentence_data = df.sentence.tolist()
df
```
![sentence_data](https://github.com/LuJH12/Weaviate-use/blob/main/figure/sentence_data.png)

在整理好数据后，我们就要把数据转成向量形式，我们先定义embeddings模型
```python
# 定义embeddings模型
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('emb_model/text2vec-large-chinese')   # embeddings模型路径
```

将数据进行向量化
```python
# 句子向量化
sentence_embeddings = model.encode(sentence_data)
# sentence_embeddings = model.encode(sentence_data)
sentence_embeddings
```
![sentence_embeddings](https://github.com/LuJH12/Weaviate-use/blob/main/figure/sentence_embeddings.png)

为了方便，我们再将`sentence_data`和`sentence_embeddings`整合到同一个DataFrame中
```python
# 将句子和embeddings后的数据整合到DataFrame里面
data = {'sentence':sentence_data,
        'embeddings':sentence_embeddings.tolist()}
df = pd.DataFrame(data)
df
```
![sentence_data_embeddings](https://github.com/LuJH12/Weaviate-use/blob/main/figure/sentence_data_embeddings.png)

在处理好数据后，我们就可以开始将数据导入Weaviate中了
```python
with client.batch(
    batch_size=100
) as batch:
    for i in range(df.shape[0]):
#         if i%20 == 0:
        print('importing data: {}'.format(i+1))
        # 定义properties
        properties = {
            'sentence_id': i+1,          # 这里是句子id, [1, 2, 3, ...]
            'sentence': df.sentence[i],  # 这里是句子内容
#             'embeddings': df.embeddings[i],
        }
        custom_vector = df.embeddings[i] # 这里是句子向量化后的数据
        # 导入数据
        client.batch.add_data_object(
            properties,
            class_name=class_name,
            vector=custom_vector
        )
print('import completed')
```

在导入数据后，就可以开始进行相似度搜索了，这里先将我们要查询的句子/词进行向量化，然后给到weaviate中，并选择返回top5个。
```python
query = model.encode(['除暴安良'])[0].tolist()   # 这里将问题进行embeddings
nearVector = {
    'vector': query
}

response = (
    client.query
    .get(class_name, ['sentence_id', 'sentence']) # 第一个参数为class名字，第二个参数为需要显示的信息
    .with_near_vector(nearVector)             # 使用向量检索，nearVector为输入问题的向量形式
    .with_limit(5)                            # 返回个数(TopK)，这里选择返回5个
    .with_additional(['distance'])            # 选择是否输出距离
    .do()
)
```

在运行代码后，我们可以看下搜索结果：
```python
print(json.dumps(response, indent=2))  # 看下输出
```
![code_output](https://github.com/LuJH12/Weaviate-use/blob/main/figure/code_output.png)

整理一下并输出，可以看到，第一句话确实有`除暴安良`这几个字
```python
# 输出结果
for i in response['data']['Get'][class_name]:
    print('='*20)
    print(i['sentence'])
```
![final_output](https://github.com/LuJH12/Weaviate-use/blob/main/figure/final_output.png)

# 总结
本教程只作简单使用。
