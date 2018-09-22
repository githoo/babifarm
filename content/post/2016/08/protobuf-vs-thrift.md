
+++
categories = ["技术"]
date = "2016-08-02T09:06:46-07:00"
draft = false
slug = ""
tags = ["protobuf","Thrift"]
title = "Protobuf vs Thrift 性能测试"

+++

 

## 简单对象结构的对比
### protobuf 文件  
```xml
package com.many.pro;
message User
{
    required string id = 1;
    required int32  weight = 2;
    required int32 age = 3;
    required string name = 4;
    required string title = 5;
    required string desc = 6;
}

```
### thrift  文件  
```xml
namespace java com.many.thrift
struct User{

        1: required string id ;
        2: required i32  weight  ;
        3: required i32 age  ;
        4: required string name  ;
        5: required string title ;
        6: required string desc ;

}
```

### 测试前提：   
测试对象：测试时都会设置相同的属性值，保证测试公平   

### 测试结果：   
1、固定1分钟测试结果
   
名称|动作|大小|次数
------|-----------|------|------
protobuf|序列化|965 byte|12,560,420
thrift  |序列化|994 byte| 8,487,338 
protobuf|反序列化|965 byte|88,119,395
thrift  |反序列化|994 byte|10,147,918 
protobuf|序列化反序列化交替进行|965 byte|10,713,643*2
thrift  |序列化反序列化交替进行|994 byte| 4,495,906*2


2、固定次数（1000000次）测试结果：

名称|动作|大小|时间(ms)
------|------------|------|------
protobuf|序列化|965 byte|4782 
thrift  |序列化|994 byte|6965 
protobuf|反序列化|965 byte|624 
thrift  |反序列化|994 byte|5662

### 测试代码 
1、protobuf 固定100000此测试代码（简单对象）
```java
package com.many.pro.thriftvsproto;

import com.google.protobuf.InvalidProtocolBufferException;
import com.many.pro.UserModel;


public class ProtoSimpleTest {
    private byte [] bytes;
    public void serialization(){
        long start = System.currentTimeMillis();
        for(int i=0;i<1000000;i++){
            UserModel.User.Builder builder = UserModel.User.newBuilder();
            builder.setId("123");
            builder.setAge(123);
            builder.setDesc("ni hao i am test");
            builder.setName("test");
            builder.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
            builder.setWeight(200);
            UserModel.User  user = builder.build();
            user.toByteArray();
        }
        long lastTime = System.currentTimeMillis() - start;
        initBytes();
        int size = bytes.length;
        System.out.println("protobuffer 序列化循环100w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public void un_serialization(){
        initBytes();
        long start = System.currentTimeMillis();
        try {
            for(int i=0;i<1000000;i++){

                    UserModel.User  user =  UserModel.User.parseFrom(bytes);

            }
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }

        int size = bytes.length;
        long lastTime = System.currentTimeMillis() - start;
        System.out.println("protobuffer 反序列化循环100w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public void initBytes(){
        UserModel.User.Builder builder = UserModel.User.newBuilder();
        builder.setId("123");
        builder.setAge(123);
        builder.setDesc("ni hao i am test");
        builder.setName("test");
        builder.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
        builder.setWeight(200);
        UserModel.User  user = builder.build();
        bytes = user.toByteArray();

    }


    public static void main(String [] args){
        ProtoSimpleTest test = new ProtoSimpleTest();
        test.serialization();
        test.un_serialization();

    }
}

```
2、thrift 固定100000此测试代码（简单对象）
```java    
package com.many.pro.thriftvsproto;


import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.transport.TIOStreamTransport;
import org.apache.thrift.transport.TTransport;
import com.many.thrift.User;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;


public class ThriftSimlpleTest {
    private byte [] bytes;
    public void serialization() throws TException {
        long start = System.currentTimeMillis();
        for(int i=0;i<1000000;i++){
            User user  = new User();
            user.setId("123");
            user.setAge(123);
            user.setDesc("ni hao i am test");
            user.setName("test");
            user.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                    "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
            user.setWeight(200);
            // 序列化
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            TTransport trans = new TIOStreamTransport(out);
            TBinaryProtocol tp = new TBinaryProtocol(trans);
            user.write(tp);
            //out.toByteArray();
        }
        long lastTime = System.currentTimeMillis() - start;
        initBytes();
        int size = bytes.length;
        System.out.println("Thrift 序列化循环100w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }
    public void un_serialization() throws TException {
        initBytes();
        long start = System.currentTimeMillis();
        for(int i=0;i<1000000;i++){

            // 反序列化
            ByteArrayInputStream in = new ByteArrayInputStream(bytes);
            TTransport trans = new TIOStreamTransport(in);
            TBinaryProtocol tp  = new TBinaryProtocol(trans);
            User user  = new User();
            user.read(tp);
        }

        int size = bytes.length;
        long lastTime = System.currentTimeMillis() - start;
        System.out.println("Thrift 反序列化循环100w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public void initBytes() throws TException {
        User user  = new User();
        user.setId("123");
        user.setAge(123);
        user.setDesc("ni hao i am test");
        user.setName("test");
        user.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
        user.setWeight(200);
        // 序列化
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        TTransport trans = new TIOStreamTransport(out);
        TBinaryProtocol tp = new TBinaryProtocol(trans);
        user.write(tp);
        bytes = out.toByteArray();

    }

    public static void main(String [] args) throws TException {
        ThriftSimlpleTest test = new ThriftSimlpleTest();
        test.serialization();
        test.un_serialization();

    }

}

```
3、protobuf 固定1分钟此测试代码（简单对象）
```java 
package com.many.pro.thriftvsproto;

import com.google.protobuf.InvalidProtocolBufferException;
import com.many.pro.UserModel;

import java.util.concurrent.*;


public class PBSimpleOneMinutes {

    static int sum = 0;

    static  final ExecutorService exec = Executors.newFixedThreadPool(1);

    private static  byte  [] bytes ;

    static class Service implements Runnable{

        @Override
        public void run() {
            while (true){
                serialization();
                un_serialization();
                sum++;
            }
        }
    }

    public static void un_serialization(){
       // initBytes();
      //  long start = System.currentTimeMillis();
        try {
             UserModel.User.parseFrom(bytes);
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }

        //int size = bytes.length;
       // long lastTime = System.currentTimeMillis() - start;
       // System.out.println("protobuffer 反序列化循环100w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public static void serialization(){
        UserModel.User.Builder builder = UserModel.User.newBuilder();
        builder.setId("123");
        builder.setAge(123);
        builder.setDesc("ni hao i am test");
        builder.setName("test");
        builder.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
        builder.setWeight(200);
        UserModel.User  user = builder.build();
        user.toByteArray();

    }


    public static void initBytes(){
        UserModel.User.Builder builder = UserModel.User.newBuilder();
        builder.setId("123");
        builder.setAge(123);
        builder.setDesc("ni hao i am test");
        builder.setName("test");
        builder.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
        builder.setWeight(200);
        UserModel.User  user = builder.build();
        bytes = user.toByteArray();

    }

    public static void main(String [] a){

        initBytes();
        Future future = exec.submit(new Service());
        try {
            future.get(60, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }

        System.out.println("60 seconds run " + sum + " tiems");
    }

}

```

4、thrift 固定1分钟此测试代码（简单对象）
```java 
package com.many.pro.thriftvsproto;



import org.apache.thrift.TException;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.util.concurrent.*;
import com.many.thrift.User;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.transport.TIOStreamTransport;
import org.apache.thrift.transport.TTransport;



public class ThriftSimpleOneMinutes {

    static int sum = 0;

    private static byte [] bytes;

    static  final ExecutorService exec = Executors.newFixedThreadPool(1);

    static class Service implements Runnable{
        @Override
        public void run() {
            while (true){
                try {
                    serialization();
                    un_serialization();
                } catch (TException e) {
                    e.printStackTrace();
                }
                sum++;
            }
        }
    }

    public static void serialization() throws TException {
        User user  = new User();
        user.setId("123");
        user.setAge(123);
        user.setDesc("ni hao i am test");
        user.setName("test");
        user.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
        user.setWeight(200);
        // 序列化
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        TTransport trans = new TIOStreamTransport(out);
        TBinaryProtocol tp = new TBinaryProtocol(trans);
        user.write(tp);
    }

    public static void un_serialization() throws TException {

        ByteArrayInputStream in = new ByteArrayInputStream(bytes);
        TTransport trans = new TIOStreamTransport(in);
        TBinaryProtocol tp  = new TBinaryProtocol(trans);
        User user  = new User();
        user.read(tp);
    }

    public static void initBytes() throws TException {
        User user  = new User();
        user.setId("123");
        user.setAge(123);
        user.setDesc("ni hao i am test");
        user.setName("test");
        user.setTitle("this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization" +
                "this is a test vs thrift serialization this is a test vs thrift serialization this is a test vs thrift serialization");
        user.setWeight(200);
        // 序列化
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        TTransport trans = new TIOStreamTransport(out);
        TBinaryProtocol tp = new TBinaryProtocol(trans);
        user.write(tp);
        bytes = out.toByteArray();

    }

    public static void main(String [] a){
        try {
            initBytes();
        } catch (TException e) {
            e.printStackTrace();
        }
        Future future = exec.submit(new Service());
        try {
            future.get(60, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }

        System.out.println("60 seconds run " + sum + " tiems");
    }

}

```
## 复杂对象结构的对比
### protobuf 文件  
```xml
// See README.txt for information and build instructions.

package tutorial;

option java_package = "com.many.pro";
option java_outer_classname = "PersonAddressBook";

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}

// Our address book file is just one of these.
message AddressBook {
  required string name = 1;
  repeated Person person = 2; // contacts
}

```

### thrift 文件  
```xml
namespace java com.many.thrift

// phone type
enum PhoneType {
  MOBILE =      0
  HOME =        1
  WORK =        2
 }

struct PhoneNumber {
    1: required string number;
    2: optional PhoneType type = PhoneType.MOBILE;
}

struct Person{
  1: required string name;
  2: required i32 id;
  3: optional string email;
  4: required list<PhoneNumber> phones;

}

struct AddressBook {
  1: required string name;
  2: required list<Person> persons;// contacts
}
```

### 测试前提：   
测试对象：测试时都会设置相同的属性值，保证测试公平  
### 测试结果：   
1、固定1分钟测试结果
   
名称|动作|大小|次数
------|-----------|------|------
protobuf|序列化|29934 byte|52345
thrift  |序列化|44476 byte|43103 
protobuf|反序列化|29934 byte|200191
thrift  |反序列化|44476 byte|44181 
protobuf|序列化反序列化交替进行|29934 byte|41788*2
thrift  |序列化反序列化交替进行|44476 byte|23028*2


2、固定次数（1000000次）测试结果：

名称|动作|大小|时间(ms)
------|------------|------|------
protobuf|序列化|29934 byte|12956
thrift  |序列化|44476 byte|16951
protobuf|反序列化|29934 byte|3120
thrift  |反序列化|44476 byte|14429

### 测试代码 
1、protobuf 固定10000次测试代码（复杂对象）
```java 
package com.many.pro.thriftvsproto;

import com.google.protobuf.InvalidProtocolBufferException;
import com.many.pro.PersonAddressBook;


public class ProtoComplexTest {
    private byte [] bytes;
    public void serialization(){
        long start = System.currentTimeMillis();
        for(int i=0;i<10000;i++){

            // book
            PersonAddressBook.AddressBook.Builder book_builder = PersonAddressBook.AddressBook.newBuilder();
            book_builder.setName("rec_system");

            //300个 person_builder
            for(int j=0;j<300;j++){
                PersonAddressBook.Person.Builder person_builder = PersonAddressBook.Person.newBuilder();
                person_builder.setId(j);
                person_builder.setName("zhangbin_" + j);
                person_builder.setEmail("zhaojun@gmail.com" +j);

                PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder =  PersonAddressBook.Person.PhoneNumber.newBuilder();
                phoneNumber_builder.setNumber("1298912313"+j);
                phoneNumber_builder.setType(PersonAddressBook.Person.PhoneType.HOME);
                PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_1 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
                phoneNumber_builder_1.setNumber("129891231we3"+j);
                phoneNumber_builder_1.setType(PersonAddressBook.Person.PhoneType.MOBILE);
                PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_2 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
                phoneNumber_builder_2.setNumber("129891231we3"+j);
                phoneNumber_builder_2.setType(PersonAddressBook.Person.PhoneType.WORK);

                person_builder.addPhone(phoneNumber_builder);
                person_builder.addPhone(phoneNumber_builder_1);
                person_builder.addPhone(phoneNumber_builder_2);

                book_builder.addPerson(person_builder);
            }

            PersonAddressBook.AddressBook  book  = book_builder.build();
            book.toByteArray();
          //  System.out.println(i);
        }
        long lastTime = System.currentTimeMillis() - start;
        initBytes();
        int size = bytes.length;
        System.out.println("protobuffer 序列化循环1 w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public void un_serialization(){
        initBytes();
        long start = System.currentTimeMillis();
        try {
            for(int i=0;i<10000;i++){

                   PersonAddressBook.AddressBook.parseFrom(bytes);

            }
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }

        int size = bytes.length;
        long lastTime = System.currentTimeMillis() - start;
        System.out.println("protobuffer 反序列化循环1 w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public void initBytes(){
        // book
        PersonAddressBook.AddressBook.Builder book_builder = PersonAddressBook.AddressBook.newBuilder();
        book_builder.setName("rec_system");

        //300个 person_builder
        for(int j=0;j<300;j++){
            PersonAddressBook.Person.Builder person_builder = PersonAddressBook.Person.newBuilder();
            person_builder.setId(j);
            person_builder.setName("zhangbin_" + j);
            person_builder.setEmail("zhaojun@gmail.com" + j);

            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder.setNumber("1298912313"+j);
            phoneNumber_builder.setType(PersonAddressBook.Person.PhoneType.HOME);
            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_1 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder_1.setNumber("129891231we3"+j);
            phoneNumber_builder_1.setType(PersonAddressBook.Person.PhoneType.MOBILE);
            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_2 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder_2.setNumber("129891231we3"+j);
            phoneNumber_builder_2.setType(PersonAddressBook.Person.PhoneType.WORK);

            person_builder.addPhone(phoneNumber_builder);
            person_builder.addPhone(phoneNumber_builder_1);
            person_builder.addPhone(phoneNumber_builder_2);

            book_builder.addPerson(person_builder);
        }

        PersonAddressBook.AddressBook  book  = book_builder.build();
        bytes = book.toByteArray();

    }


    public static void main(String [] args){
        ProtoComplexTest test = new ProtoComplexTest();
        test.serialization();
        test.un_serialization();

    }
}

```
2、thrift 固定10000次测试代码（复杂对象）
```java 
package com.many.pro.thriftvsproto;



import com.many.thrift.*;
import com.many.thrift.AddressBook;
import com.many.thrift.Person;
import com.many.thrift.PhoneNumber;
import com.many.thrift.PhoneType;
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.transport.TIOStreamTransport;
import org.apache.thrift.transport.TTransport;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;


public class ThriftComplexTest {

    private byte [] bytes;



    public void serialization() throws TException {
        long start = System.currentTimeMillis();
        for(int i=0;i<10000;i++){
            AddressBook book = new AddressBook();
            book.setName("rec_system");
            // person 300
            for(int j=0;j<300;j++){
                Person person = new Person();
                person.setEmail("zhaojun@gmail.com"+j);
                person.setId(i);
                person.setName("zhangbin_" + j);

                PhoneNumber n1 = new PhoneNumber();
                n1.setNumber("1298912313"+j);
                n1.setType(PhoneType.HOME);
                PhoneNumber n2 = new PhoneNumber();
                n2.setNumber("129891231we3"+j);
                n2.setType(PhoneType.MOBILE);
                PhoneNumber n3 = new PhoneNumber();
                n3.setNumber("129891231we3" +j);
                n3.setType(PhoneType.WORK);
                person.addToPhones(n1);
                person.addToPhones(n2);
                person.addToPhones(n3);
                book.addToPersons(person);
            }

            ByteArrayOutputStream out = new ByteArrayOutputStream();
            TTransport trans = new TIOStreamTransport(out);
            TBinaryProtocol tp = new TBinaryProtocol(trans);
            book.write(tp);
        }
        long lastTime = System.currentTimeMillis() - start;
        initBytes();
        int size = bytes.length;
        System.out.println("Thrift 序列化循环1w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }



    public void un_serialization() throws TException {
        initBytes();
        long start = System.currentTimeMillis();
        for(int i=0;i<10000;i++){
            ByteArrayInputStream in = new ByteArrayInputStream(bytes);
            TTransport trans = new TIOStreamTransport(in);
            TBinaryProtocol tp  = new TBinaryProtocol(trans);
            AddressBook book  = new AddressBook();
            book.read(tp);
        }

        int size = bytes.length;
        long lastTime = System.currentTimeMillis() - start;
        System.out.println("Thrift 反序列化循环1w times，单条大小："+size+" byte"+" 耗时："+lastTime+" ms");
    }

    public void initBytes() throws TException {

        AddressBook book = new AddressBook();
        book.setName("rec_system");
        // person 300
        for(int j=0;j<300;j++){
            Person person = new Person();
            person.setEmail("zhaojun@gmail.com"+j);
            person.setId(j);
            person.setName("zhangbin_" + j);

            PhoneNumber n1 = new PhoneNumber();
            n1.setNumber("1298912313"+j);
            n1.setType(PhoneType.HOME);
            PhoneNumber n2 = new PhoneNumber();
            n2.setNumber("129891231we3"+j);
            n2.setType(PhoneType.MOBILE);
            PhoneNumber n3 = new PhoneNumber();
            n3.setNumber("129891231we3" +j);
            n3.setType(PhoneType.WORK);
            person.addToPhones(n1);
            person.addToPhones(n2);
            person.addToPhones(n3);
            book.addToPersons(person);
        }

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        TTransport trans = new TIOStreamTransport(out);
        TBinaryProtocol tp = new TBinaryProtocol(trans);
        book.write(tp);
        bytes = out.toByteArray();
    }


    public static void main(String [] args) throws TException {
        ThriftComplexTest test = new ThriftComplexTest();
        test.serialization();
        test.un_serialization();

    }

}


```

3、protobuf 固定1分钟测试代码（复杂对象）
```java 
package com.many.pro.thriftvsproto;

import com.google.protobuf.InvalidProtocolBufferException;
import com.many.pro.PersonAddressBook;
import com.many.pro.UserModel;

import java.util.concurrent.*;


public class PBComplexOneMinutes {

    static int sum = 0;

    static  final ExecutorService exec = Executors.newFixedThreadPool(1);

    private static  byte  [] bytes ;

    static class Service implements Runnable{

        @Override
        public void run() {
            while (true){
                serialization();
                un_serialization();
                sum++;
            }
        }
    }

    public static void un_serialization(){
        try {
            PersonAddressBook.AddressBook.parseFrom(bytes);
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }
    }

    public static void serialization(){
        // book
        PersonAddressBook.AddressBook.Builder book_builder = PersonAddressBook.AddressBook.newBuilder();
        book_builder.setName("rec_system");

        //300个 person_builder
        for(int j=0;j<300;j++){
            PersonAddressBook.Person.Builder person_builder = PersonAddressBook.Person.newBuilder();
            person_builder.setId(j);
            person_builder.setName("zhangbin_" + j);
            person_builder.setEmail("zhaojun@gmail.com" +j);

            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder.setNumber("1298912313"+j);
            phoneNumber_builder.setType(PersonAddressBook.Person.PhoneType.HOME);
            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_1 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder_1.setNumber("129891231we3"+j);
            phoneNumber_builder_1.setType(PersonAddressBook.Person.PhoneType.MOBILE);
            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_2 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder_2.setNumber("129891231we3"+j);
            phoneNumber_builder_2.setType(PersonAddressBook.Person.PhoneType.WORK);

            person_builder.addPhone(phoneNumber_builder);
            person_builder.addPhone(phoneNumber_builder_1);
            person_builder.addPhone(phoneNumber_builder_2);

            book_builder.addPerson(person_builder);
        }

        PersonAddressBook.AddressBook  book  = book_builder.build();
        book.toByteArray();

    }


    public static void initBytes(){
        // book
        PersonAddressBook.AddressBook.Builder book_builder = PersonAddressBook.AddressBook.newBuilder();
        book_builder.setName("rec_system");

        //300个 person_builder
        for(int j=0;j<300;j++){
            PersonAddressBook.Person.Builder person_builder = PersonAddressBook.Person.newBuilder();
            person_builder.setId(j);
            person_builder.setName("zhangbin_" + j);
            person_builder.setEmail("zhaojun@gmail.com" + j);

            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder.setNumber("1298912313"+j);
            phoneNumber_builder.setType(PersonAddressBook.Person.PhoneType.HOME);
            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_1 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder_1.setNumber("129891231we3"+j);
            phoneNumber_builder_1.setType(PersonAddressBook.Person.PhoneType.MOBILE);
            PersonAddressBook.Person.PhoneNumber.Builder phoneNumber_builder_2 =  PersonAddressBook.Person.PhoneNumber.newBuilder();
            phoneNumber_builder_2.setNumber("129891231we3"+j);
            phoneNumber_builder_2.setType(PersonAddressBook.Person.PhoneType.WORK);

            person_builder.addPhone(phoneNumber_builder);
            person_builder.addPhone(phoneNumber_builder_1);
            person_builder.addPhone(phoneNumber_builder_2);

            book_builder.addPerson(person_builder);
        }

        PersonAddressBook.AddressBook  book  = book_builder.build();
        bytes = book.toByteArray();

    }

    public static void main(String [] a){

        initBytes();
        Future future = exec.submit(new Service());
        try {
            future.get(60, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }

        System.out.println("60 seconds run " + sum + " tiems");
    }

}

```


4、thrift 固定1分钟测试代码（复杂对象）
```java 
package com.many.pro.thriftvsproto;


import com.many.thrift.AddressBook;
import com.many.thrift.Person;
import com.many.thrift.PhoneNumber;
import com.many.thrift.PhoneType;
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.transport.TIOStreamTransport;
import org.apache.thrift.transport.TTransport;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.util.concurrent.*;



public class ThriftComplexOneMinutes {

    static int sum = 0;

    private static byte [] bytes;

    static  final ExecutorService exec = Executors.newFixedThreadPool(1);

    static class Service implements Runnable{

        @Override
        public void run() {
            while (true){
                try {
                    serialization();
                   /// un_serialization();
                } catch (TException e) {
                    e.printStackTrace();
                }
                sum++;
            }
        }
    }

    public static void serialization() throws TException {
        AddressBook book = new AddressBook();
        book.setName("rec_system");
        // person 300
        for(int j=0;j<300;j++){
            Person person = new Person();
            person.setEmail("zhaojun@gmail.com"+j);
            person.setId(j);
            person.setName("zhangbin_" + j);

            PhoneNumber n1 = new PhoneNumber();
            n1.setNumber("1298912313"+j);
            n1.setType(com.many.thrift.PhoneType.HOME);
            PhoneNumber n2 = new PhoneNumber();
            n2.setNumber("129891231we3"+j);
            n2.setType(PhoneType.MOBILE);
            PhoneNumber n3 = new PhoneNumber();
            n3.setNumber("129891231we3" +j);
            n3.setType(PhoneType.WORK);
            person.addToPhones(n1);
            person.addToPhones(n2);
            person.addToPhones(n3);
            book.addToPersons(person);
        }

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        TTransport trans = new TIOStreamTransport(out);
        TBinaryProtocol tp = new TBinaryProtocol(trans);
        book.write(tp);
    }

    public static void un_serialization() throws TException {

        ByteArrayInputStream in = new ByteArrayInputStream(bytes);
        TTransport trans = new TIOStreamTransport(in);
        TBinaryProtocol tp  = new TBinaryProtocol(trans);
        AddressBook book  = new AddressBook();
        book.read(tp);
    }

    public static void initBytes() throws TException {
        AddressBook book = new AddressBook();
        book.setName("rec_system");
        // person 300
        for(int j=0;j<300;j++){
            Person person = new Person();
            person.setEmail("zhaojun@gmail.com"+j);
            person.setId(j);
            person.setName("zhangbin_" + j);

            PhoneNumber n1 = new PhoneNumber();
            n1.setNumber("1298912313"+j);
            n1.setType(PhoneType.HOME);
            PhoneNumber n2 = new PhoneNumber();
            n2.setNumber("129891231we3"+j);
            n2.setType(PhoneType.MOBILE);
            PhoneNumber n3 = new PhoneNumber();
            n3.setNumber("129891231we3" +j);
            n3.setType(PhoneType.WORK);
            person.addToPhones(n1);
            person.addToPhones(n2);
            person.addToPhones(n3);
            book.addToPersons(person);
        }

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        TTransport trans = new TIOStreamTransport(out);
        TBinaryProtocol tp = new TBinaryProtocol(trans);
        book.write(tp);
        bytes = out.toByteArray();

    }

    public static void main(String [] a){
        try {
            initBytes();
        } catch (TException e) {
            e.printStackTrace();
        }
        Future future = exec.submit(new Service());
        try {
            future.get(60, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }

        System.out.println("60 seconds run " + sum + " tiems");
    }

}

```