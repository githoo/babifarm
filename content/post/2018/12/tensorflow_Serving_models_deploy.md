+++
categories = ["技术"]
date = "2018-12-15T10:06:46-07:00"
draft = false
slug = ""
tags = ["core"]
title = "TFServing models deploy"
+++


## Tensorflow Serving 多模型部署

### 1、单模型方式

#### 多个单模型启动：

```
#单模型
root_path='/export/mtfs/TF'
docker run -p 10004:8501 -p 10005:8500 -v $root_path/corepw_model:/models/corepw_model -e MODEL_NAME=corepw_model -t tensorflow/serving &> my_log &

docker run -p 10006:8501 -p 10007:8500 -v $root_path/brand_model:/models/brand_model -e MODEL_NAME=brand_model -t tensorflow/serving &> my_log1 &



```

#### 访问示例：

```
#访问例子
curl -H "Content-Type:application/json" -X POST  http://localhost:10004/v1/models/corepw_model:predict -d '
> {
>     "instances": [{
>         "input_sentence_lengths": 37,
>         "input_sentences": [86, 1535, 0, 0, 0, 0, 0, 0, 0, 73, 41, 4, 0, 10, 40, 368, 1888, 73, 41, 492, 86, 764, 202, 74, 96, 67, 447, 447, 39, 0, 8, 173, 29, 1, 758, 7, 41]
>     }],
>     "signature_name": "predict_tags"
> } '

{
    "predictions": [[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    ]
}


curl -H "Content-Type:application/json" -X POST  http://localhost:10006/v1/models/brand_model:predict -d '
{
    "instances": [{
        "input_sentence_lengths": 37,
        "input_sentences": [86, 1535, 0, 0, 0, 0, 0, 0, 0, 73, 41, 4, 0, 10, 40, 368, 1888, 73, 41, 492, 86, 764, 202, 74, 96, 67, 447, 447, 39, 0, 8, 173, 29, 1, 758, 7, 41]
    }],
    "signature_name": "predict_tags"
} '

{
    "predictions": [[1, 2, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    ]
}

```





### 2、多模型方式

#### 多模型配置：

```
#配置文件
#/models/models.conf 

model_config_list: {
  config: {
    name: "corepw_model",
    base_path: "/models/corepw_model",
    model_platform: "tensorflow"
  },
  config: {
    name: "brand_model",
    base_path: "/models/brand_model",
    model_platform: "tensorflow"
  }
}

```

#### 启动多模型：

```
#多模型docker部署步骤

containerid=`docker container ls |awk '{print $1}'|sed -n '2p'`
docker container rm -f $containerid
docker image rm tfserving_online

docker run -d --name serving_base tensorflow/serving
root_path='/export/mtfs/TF'
docker cp $root_path/corepw_model serving_base:/models/corepw_model
docker cp $root_path/brand_model serving_base:/models/brand_model
docker cp $root_path/models.conf serving_base:/models/models.conf
docker commit serving_base tfserving_online
containerid=`docker container ls |awk '{print $1}'|sed -n '2p'`
docker container rm -f $containerid

docker run -p 10002:8501 -p 10003:8500 -t tfserving_online --model_config_file=/models/models.conf

```

#### 访问示例：

```
#访问例子
curl -H "Content-Type:application/json" -X POST  http://localhost:10002/v1/models/corepw_model:predict -d '
{
    "instances": [{
        "input_sentence_lengths": 37,
        "input_sentences": [86, 1535, 0, 0, 0, 0, 0, 0, 0, 73, 41, 4, 0, 10, 40, 368, 1888, 73, 41, 492, 86, 764, 202, 74, 96, 67, 447, 447, 39, 0, 8, 173, 29, 1, 758, 7, 41]
    }],
    "signature_name": "predict_tags"
} '


curl -H "Content-Type:application/json" -X POST  http://localhost:10002/v1/models/brand_model:predict -d '
{
    "instances": [{
        "input_sentence_lengths": 37,
        "input_sentences": [86, 1535, 0, 0, 0, 0, 0, 0, 0, 73, 41, 4, 0, 10, 40, 368, 1888, 73, 41, 492, 86, 764, 202, 74, 96, 67, 447, 447, 39, 0, 8, 173, 29, 1, 758, 7, 41]
    }],
    "signature_name": "predict_tags"
} '

```



### 3、压测注意事项

```
QPS的影响因素：

1、网络参数，考虑模型可以接受的准确率的情况下，尽量调小模型参数；

2、输入数据长度，输入数据转换向量时使用自适应长度，减少不必要的计算；

```

