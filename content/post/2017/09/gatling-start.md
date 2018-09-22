+++
categories = ["工具"]
date = "2017-09-09T10:06:46-07:00"
draft = false
slug = ""
tags = ["test","gatling"]
title = "gatling start"

+++

## gatlint start

- gatling quick start


http://gatling.io/docs/current/quickstart/

- 源码： 

   https://github.com/gatling/gatling#wiki-loops

- gatling下载地址：
  https://repo1.maven.org/maven2/io/gatling/highcharts/gatling-charts-highcharts-bundle/2.2.5/gatling-charts-highcharts-bundle-2.2.5-bundle.zip

- gatling目录结构：
  * bin目录下有2个脚本，gatling和recorder， gatling用来运行测试， recorder用来启动录制脚本的UI的（不推荐使用），
  * conf目录是关于Gatling自身的一些配置。
  * lib目录是Gatling自身依赖的库文件。
  * results目录用来存放测试报告的。
  * user-files目录是用来存放测试脚本的。
  * 当运行gating脚本的时候，其会扫描user-files目录下的所有文件，列出其中所有的Simulation(一个测试类，里面可以包含任意多个测试场景)

- 执行压测：

  执行gatling时，可通过参数制定具体执行那个实例，如下：
  ./bin/gatling.sh -s scala文件的class名字
  比如./bin/gatling.sh -s TestGet

- 录制脚本：
  * 需要配置Internet的代理才能录制成功。录制完成之后代码保存在user-files/simulations/ 目录下
  * 配置浏览器代理：https://my.oschina.net/u/2419564/blog/613048



- 压测QPS和持续时间设置：

QPS为1 持续60秒
setUp(
        recall.inject(constantUsersPerSec(1) during (60 seconds))
    ).protocols(httpProtocol)

QPS为10000持续一小时
setUp(        recall.inject(constantUsersPerSec(10000) during (3600 seconds))
    ).protocols(httpProtocol)
​    

- Gatling的报表：


GLOBAL和DETAILS，其中GLOBAL主要是请求相关的统计数据，比如每秒请求数，请求成功与失败数等；其中DETAILS主要是请求时间相关的统计数据，比如请求响应时间，请求响应延迟时间等

- 官方指南：


[http://gatling.io/docs/current/](http://gatling.io/docs/current/)

*User’s guide**

  [What’s New](http://gatling.io/docs/current/whats_new)

  [Migration Guides](http://gatling.io/docs/current/migration_guides)

  [Quickstart](http://gatling.io/docs/current/quickstart)

  [Advanced Tutorial](http://gatling.io/docs/current/advanced_tutorial)

  [General](http://gatling.io/docs/current/general)

  [Session](http://gatling.io/docs/current/session)

  [HTTP](http://gatling.io/docs/current/http)

  [JMS](http://gatling.io/docs/current/jms)

  [Realtime monitoring](http://gatling.io/docs/current/realtime_monitoring)

  [Extensions](http://gatling.io/docs/current/extensions)

  [Cookbook](http://gatling.io/docs/current/cookbook)

  [Information for Gatling Developers](http://gatling.io/docs/current/developing_gatling)

  [Project Information](http://gatling.io/docs/current/project)

- 使用实例：


下面主要通过一些压测实例介绍gatling的基本使用
1、post 方式压测：
```
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import io.gatling.jdbc.Predef._
class TestPost extends Simulation {
  val httpProtocol = http.baseURL("http://10.188.188.88:8088/")
  //注意这里,设置提交内容type
  val headers_json = Map("Content-Type" -> "application/json")
  val scn = scenario("test post") 
    .exec(http("test_post")   //http 请求name
      .post("test")     //post url
      .headers(headers_json)  //设置body数据格式
      //将json参数用StricngBody包起,并作为参数传递给function body()
      .body(StringBody("{\"content\":\"改革开放三十年来，在祖国的逐步强大之下，我们快乐的成长\"}")).asJSON)
  
  setUp(
        scn.inject(constantUsersPerSec(1000) during (5 seconds))
    ).protocols(httpProtocol)

}
```

2、带参数的get方式压测：
```
import scala.concurrent.duration._
import scala.collection.mutable.ArrayBuffer
import scala.io.Source

import io.gatling.core.Predef._
import io.gatling.http.Predef._
import io.gatling.jdbc.Predef._

class TestGet extends Simulation {
	
	object GetData {
      		val feeder=tsv("sku-tag.csv").circular
      		val recall = feed(feeder)
        		.exec(http("test")
	    		.get("/test?sku=${sku}&tagid=${tagid}&pin=joy&pid=20001&lid=111&uuid=8888"))
    	}

	val httpProtocol = http.baseURL("http://10.188.188.88:8088")
		.inferHtmlResources()
		.acceptHeader("application/json;q=0.9,*/*;q=0.8")
		.acceptEncodingHeader("gzip, deflate")
		.acceptLanguageHeader("en-US,en;q=0.5")
		.doNotTrackHeader("1")
        .disableFollowRedirect
		.userAgentHeader("Mozilla/5.0 (Windows NT 6.1; WOW64; rv:39.0) Gecko/20100101 Firefox/39.0")
	val headers_0 = Map(
		"Pragma" -> "no-cache",
		"Upgrade-Insecure-Requests" -> "1")

	val test = scenario("test").exec(GetData.recall)
	setUp(
        	test.inject(constantUsersPerSec(9000) during (600 seconds))
    	).protocols(httpProtocol) 
}
```
sku-tag.csv格式
```
sku	tagid
1002880	2d9b98dbba99f729
1002880	e73391bf8ddda234
1002880	cdb704084604e664
1002880	e50b8ea837b42365
1003403	f8952a488a11ced2
1003403	e2c719d0c61dc7a7
1003403	6fcb75f7dcc4012b
1003403	e73391bf8ddda234
```


3、数据库方式get压测：
```
import scala.concurrent.duration._
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import io.gatling.jdbc.Predef._
import java.sql.Timestamp
import java.text.SimpleDateFormat
import java.util.Date
import scala.util.Random
class TestDB extends Simulation {

    object GetData {
      val format = new SimpleDateFormat("yyyy-MM-dd")
      val rnd = new Random
      var flag = rnd.nextInt(18)
      val feeder=jdbcFeeder("jdbc:mysql://10.188.188.136:3306/test", "root", "", "select url,cookie from test.test_data limit "+flag*2000000 +",2000000").circular
      
      val recall = feed(feeder)
        .exec(http("Search")        
		.get("${url}")
        .header("Cookie", "${cookie}")
		.check(status.is(200)))        
    }

	val httpProtocol = http	      
        .baseURL("http://10.188.188.88:8088")
		//.inferHtmlResources()
		.acceptHeader("application/json;q=0.9,*/*;q=0.8")
		.acceptEncodingHeader("gzip, deflate")
		.acceptLanguageHeader("en-US,en;q=0.5")
		//.connection("keep-alive")
		.doNotTrackHeader("1")
        .disableFollowRedirect
		.userAgentHeader("Mozilla/5.0 (Windows NT 6.1; WOW64; rv:39.0) Gecko/20100101 Firefox/39.0")
		
	val test = scenario("test").exec(GetData.recall)

	setUp(
		test.inject(constantUsersPerSec(10000) during (36000 seconds))
	).protocols(httpProtocol)
}
```

