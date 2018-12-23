+++
categories = ["技术"]
date = "2018-12-23T10:06:46-07:00"
draft = false
slug = ""
tags = ["core"]
title = "TFServing model load test"
+++




## Tensorflow Serving  model  test



#### Tensorflow model

```
#Half Plus Two test tfserving-gpu

#from :https://github.com/tensorflow/serving/blob/master/tensorflow_serving/g3doc/docker.md

mkdir -p /tmp/tfserving
cd /tmp/tfserving
git clone https://github.com/tensorflow/serving

nvidia-docker run -p 18501:8501 \
  -v /tmp/tfserving/serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu:/models/half_plus_two \
  -e MODEL_NAME=half_plus_two -t tensorflow/serving:latest-gpu &

curl -d '{"instances": [1.0, 2.0, 5.0]}' \
  -X POST http://localhost:18501/v1/models/half_plus_two:predict

curl -H "Content-Type:application/json" -X POST --data '{"instances": [1.0, 2.0, 5.0]}' -X POST http://localhost:18501/v1/models/half_plus_two:predict
```

#### gatling loadtest

```
import io.gatling.core.Predef._
import io.gatling.core.scenario.Simulation
import io.gatling.http.Predef._
import scala.concurrent.duration._
class x_tfsgpuN extends Simulation {
  val httpConf = http.baseURL("http://localhost:18501/v1/models")
  //注意这里,设置提交内容type
  val data="{\"instances\":[1.0, 2.0, 5.0]}"
  
  val headers_json = Map("Content-Type" -> "application/json")
  val scn = scenario("ExractPW")
    .exec(http("simple")   //http 请求name
      .post("/half_plus_two:predict")     //post url
      .headers(headers_json)  //设置body数据格式
      //将json参数用StringBody包起,并作为参数传递给function body()
      .body(StringBody(data)).asJSON)
  setUp(scn.inject(constantUsersPerSec(20000) during(60 seconds))).protocols(httpConf)
}

```

