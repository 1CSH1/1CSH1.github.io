---
title: Json 工具类
date: 2017-04-05 08:00:00
categories: [Java, Json]
tags: [Java, Json]
---

个人博客原文：[Json 工具类](https://1csh1.github.io/2017/04/05/json-util/)
代码：[JsonUtil.java](https://github.com/1CSH1/util/blob/master/src/main/java/com/example/util/json/JsonUtil.java)

> 摘要：文中使用了阿里的Json转换工具fastjson来做封装，实现了对象和Json互转，对象数组和Json互转，list和Json互转，Map和Json互转。在项目中可以直接使用该工具类来做Json和其他对象的转换。

# maven
```
<!-- fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.30</version>
</dependency>
```


# 工具类代码：
JsonUtil.java

```
package com.example.util.json;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.JSONLibDataFormatSerializer;
import com.alibaba.fastjson.serializer.SerializeConfig;
import com.alibaba.fastjson.serializer.SerializerFeature;

import java.util.List;
import java.util.Map;
import java.util.Objects;

/**
 * @author James-CSH
 * @since 17-3-29 下午10:41
 */
public class JsonUtil {

    private static final SerializeConfig config;

    static {
        config = new SerializeConfig();
        // compatible with the java.util.Date and the java.sql.Date
        config.put(java.util.Date.class, new JSONLibDataFormatSerializer());
        config.put(java.sql.Date.class, new JSONLibDataFormatSerializer());
    }

    /**
     * set the default value of the object which the value is null
     */
    private static final SerializerFeature[] features = {
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.WriteNullListAsEmpty,
            SerializerFeature.WriteNullBooleanAsFalse,
            SerializerFeature.WriteNullStringAsEmpty,
    };

    /**
     * list to json
     *
     * @param list
     *          the list that will transform to json string
     * @return
     *          the json string of list transform
     */
    public static String list2json(List list) {
        return JSON.toJSONString(list);
    }

    /**
     * map to json
     * @param map
     *          the map that will transform to json string
     * @return
     *          the json string of map transform
     */
    public static String map2json(Map map) {
        return JSONObject.toJSONString(map);
    }

    /**
     * object array to json
     *
     * @param objects
     *          the object array that will transform to json string
     * @return
     *          the json string of array transform
     */
    public static String array2json(Object[] objects) {
        return JSON.toJSONString(objects);
    }

    /**
     * object to json
     *
     * @param object
     *          the object that will transform to json string
     * @return
     *          the json string of object
     */
    public static String object2json(Object object) {
        return JSON.toJSONString(object, config, features);
    }


    /**
     * json to list
     *
     * @param json
     *          the json string that will transform to list
     * @param clazz
     *          the class of the list's element
     * @param <T>
     *          the generic of the class
     * @return
     *          the list that json string transform
     */
    public static <T> List<T> json2list(String json, Class<T> clazz) {
        return JSON.parseArray(json, clazz);
    }

    /**
     * json to map
     *
     * @param json
     *          json string that will transform to map
     * @return
     *          the map fo json string
     */
    public static Map json2map(String json) {
        return JSONObject.parseObject(json);
    }


    /**
     * json string to object array
     *
     * @param json
     *          the json string will transform to object array
     * @param clazz
     *          the class of the json will transform
     * @param ts
     *          the real object array
     * @param <T>
     *          the real object
     * @return
     *          the object array of the json string
     *
     * @param json
     * @param clazz
     * @param ts
     * @param <T>
     * @return
     */
    public static <T> T[] json2array(String json, Class<T> clazz, T[] ts) {
        return JSON.parseArray(json, clazz).toArray(ts);
    }

    /**
     * json string to object
     *
     * @param json
     *          the json string that will transform to object
     * @param clazz
     *          the class that json will transform
     * @param <T>
     *          the object class
     * @return
     *          the object of json string
     */
    public static <T> Object json2object(String json, Class<T> clazz) {
        return JSON.parseObject(json, clazz);
    }

}
```

# 测试代码：
TestJsonUtil.java

```
package com.example.util.json;

import org.junit.Test;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author James-CSH
 * @since 17-3-31 下午10:47
 */
public class TestJsonUtil {

    @Test
    public void testObject() {
        Person person = new Person();
        person.setId(1);
        person.setName("James陈");
        person.setAge(23);
        person.setGender(1);

        Person child = new Person();
        child.setId(2);
        child.setName("James陈 儿子");
        child.setAge(2);
        child.setGender(1);

        List<Person> children = new ArrayList<Person>();
        children.add(child);
        person.setChildren(children);

        String personsJson = JsonUtil.object2json(person);
System.out.println(personsJson); //{"age":23,"children":[{"age":2,"children":[],"gender":1,"id":2,"name":"James陈 儿子"}],"gender":1,"id":1,"name":"James陈"}
        Person person2 = (Person) JsonUtil.json2object(personsJson, Person.class);
System.out.println(person2.toString()); //Person{id=1, name='James陈', age=23, gender=1, children=[Person{id=2, name='James陈 儿子', age=2, gender=1, children=[]}]}
    }

    @Test
    public void testArray() {
        Person[] persons = new Person[10];
        Person person;
        Person child;
        for (int i=0; i<persons.length; i++) {
            person = new Person();
            person.setId(i);
            person.setName("James陈" + i);
            person.setAge(20 + i);
            person.setGender(i > 5 ? 1 : 0);

            child = new Person();
            child.setId(100 + i);
            child.setName("James陈 " + i + "儿子");
            child.setAge(i);
            child.setGender(i < 5 ? 1 : 0);

            List<Person> children = new ArrayList<Person>(1);
            children.add(child);
            person.setChildren(children);

            persons[i] = person;
        }

        String personsJson = JsonUtil.array2json(persons);
System.out.println(personsJson);
//[{"age":20,"children":[{"age":0,"gender":1,"id":100,"name":"James陈 0儿子"}],"gender":0,"id":0,"name":"James陈0"},{"age":21,"children":[{"age":1,"gender":1,"id":101,"name":"James陈 1儿子"}],"gender":0,"id":1,"name":"James陈1"},{"age":22,"children":[{"age":2,"gender":1,"id":102,"name":"James陈 2儿子"}],"gender":0,"id":2,"name":"James陈2"},{"age":23,"children":[{"age":3,"gender":1,"id":103,"name":"James陈 3儿子"}],"gender":0,"id":3,"name":"James陈3"},{"age":24,"children":[{"age":4,"gender":1,"id":104,"name":"James陈 4儿子"}],"gender":0,"id":4,"name":"James陈4"},{"age":25,"children":[{"age":5,"gender":0,"id":105,"name":"James陈 5儿子"}],"gender":0,"id":5,"name":"James陈5"},{"age":26,"children":[{"age":6,"gender":0,"id":106,"name":"James陈 6儿子"}],"gender":1,"id":6,"name":"James陈6"},{"age":27,"children":[{"age":7,"gender":0,"id":107,"name":"James陈 7儿子"}],"gender":1,"id":7,"name":"James陈7"},{"age":28,"children":[{"age":8,"gender":0,"id":108,"name":"James陈 8儿子"}],"gender":1,"id":8,"name":"James陈8"},{"age":29,"children":[{"age":9,"gender":0,"id":109,"name":"James陈 9儿子"}],"gender":1,"id":9,"name":"James陈9"}]
        Person[] persons2 = new Person[]{};
        persons2 = JsonUtil.json2array(personsJson, Person.class, persons2);
        for (int i=0; i<persons2.length; i++) {
System.out.println(persons2[i].toString());
/*
Person{id=0, name='James陈0', age=20, gender=0, children=[Person{id=100, name='James陈 0儿子', age=0, gender=1, children=null}]}
Person{id=1, name='James陈1', age=21, gender=0, children=[Person{id=101, name='James陈 1儿子', age=1, gender=1, children=null}]}
Person{id=2, name='James陈2', age=22, gender=0, children=[Person{id=102, name='James陈 2儿子', age=2, gender=1, children=null}]}
Person{id=3, name='James陈3', age=23, gender=0, children=[Person{id=103, name='James陈 3儿子', age=3, gender=1, children=null}]}
Person{id=4, name='James陈4', age=24, gender=0, children=[Person{id=104, name='James陈 4儿子', age=4, gender=1, children=null}]}
Person{id=5, name='James陈5', age=25, gender=0, children=[Person{id=105, name='James陈 5儿子', age=5, gender=0, children=null}]}
Person{id=6, name='James陈6', age=26, gender=1, children=[Person{id=106, name='James陈 6儿子', age=6, gender=0, children=null}]}
Person{id=7, name='James陈7', age=27, gender=1, children=[Person{id=107, name='James陈 7儿子', age=7, gender=0, children=null}]}
Person{id=8, name='James陈8', age=28, gender=1, children=[Person{id=108, name='James陈 8儿子', age=8, gender=0, children=null}]}
Person{id=9, name='James陈9', age=29, gender=1, children=[Person{id=109, name='James陈 9儿子', age=9, gender=0, children=null}]}
 */
        }
    }

    @Test
    public void testList() {
        List<Person> persons = new ArrayList<Person>(10);
        Person person;
        Person child;
        for (int i=0; i<10; i++) {
            person = new Person();
            person.setId(i);
            person.setName("James陈" + i);
            person.setAge(20 + i);
            person.setGender(i > 5 ? 1 : 0);

            child = new Person();
            child.setId(100 + i);
            child.setName("James陈 " + i + "儿子");
            child.setAge(i);
            child.setGender(i < 5 ? 1 : 0);

            List<Person> children = new ArrayList<Person>(1);
            children.add(child);
            person.setChildren(children);

            persons.add(person);
        }

        String personsJson = JsonUtil.list2json(persons);
System.out.println(personsJson);
//[{"age":20,"children":[{"age":0,"gender":1,"id":100,"name":"James陈 0儿子"}],"gender":0,"id":0,"name":"James陈0"},{"age":21,"children":[{"age":1,"gender":1,"id":101,"name":"James陈 1儿子"}],"gender":0,"id":1,"name":"James陈1"},{"age":22,"children":[{"age":2,"gender":1,"id":102,"name":"James陈 2儿子"}],"gender":0,"id":2,"name":"James陈2"},{"age":23,"children":[{"age":3,"gender":1,"id":103,"name":"James陈 3儿子"}],"gender":0,"id":3,"name":"James陈3"},{"age":24,"children":[{"age":4,"gender":1,"id":104,"name":"James陈 4儿子"}],"gender":0,"id":4,"name":"James陈4"},{"age":25,"children":[{"age":5,"gender":0,"id":105,"name":"James陈 5儿子"}],"gender":0,"id":5,"name":"James陈5"},{"age":26,"children":[{"age":6,"gender":0,"id":106,"name":"James陈 6儿子"}],"gender":1,"id":6,"name":"James陈6"},{"age":27,"children":[{"age":7,"gender":0,"id":107,"name":"James陈 7儿子"}],"gender":1,"id":7,"name":"James陈7"},{"age":28,"children":[{"age":8,"gender":0,"id":108,"name":"James陈 8儿子"}],"gender":1,"id":8,"name":"James陈8"},{"age":29,"children":[{"age":9,"gender":0,"id":109,"name":"James陈 9儿子"}],"gender":1,"id":9,"name":"James陈9"}]
        List<Person> persons2 = JsonUtil.json2list(personsJson, Person.class);
        for (int i=0; i<persons2.size(); i++) {
System.out.println(persons2.get(i).toString());
/*
Person{id=0, name='James陈0', age=20, gender=0, children=[Person{id=100, name='James陈 0儿子', age=0, gender=1, children=null}]}
Person{id=1, name='James陈1', age=21, gender=0, children=[Person{id=101, name='James陈 1儿子', age=1, gender=1, children=null}]}
Person{id=2, name='James陈2', age=22, gender=0, children=[Person{id=102, name='James陈 2儿子', age=2, gender=1, children=null}]}
Person{id=3, name='James陈3', age=23, gender=0, children=[Person{id=103, name='James陈 3儿子', age=3, gender=1, children=null}]}
Person{id=4, name='James陈4', age=24, gender=0, children=[Person{id=104, name='James陈 4儿子', age=4, gender=1, children=null}]}
Person{id=5, name='James陈5', age=25, gender=0, children=[Person{id=105, name='James陈 5儿子', age=5, gender=0, children=null}]}
Person{id=6, name='James陈6', age=26, gender=1, children=[Person{id=106, name='James陈 6儿子', age=6, gender=0, children=null}]}
Person{id=7, name='James陈7', age=27, gender=1, children=[Person{id=107, name='James陈 7儿子', age=7, gender=0, children=null}]}
Person{id=8, name='James陈8', age=28, gender=1, children=[Person{id=108, name='James陈 8儿子', age=8, gender=0, children=null}]}
Person{id=9, name='James陈9', age=29, gender=1, children=[Person{id=109, name='James陈 9儿子', age=9, gender=0, children=null}]}
 */
        }
    }

    @Test
    public void testMap() {
        Person person = new Person();
        person.setId(1);
        person.setName("James陈");
        person.setAge(23);
        person.setGender(1);

        Person child = new Person();
        child.setId(2);
        child.setName("James陈 儿子");
        child.setAge(2);
        child.setGender(1);

        List<Person> children = new ArrayList<Person>();
        children.add(child);
        person.setChildren(children);

        Map<String, Person> map1 = new HashMap<String, Person>();
        map1.put("key1", person);
        String personJson = JsonUtil.map2json(map1);
System.out.println(personJson);
//{"key1":{"age":23,"children":[{"age":2,"gender":1,"id":2,"name":"James陈 儿子"}],"gender":1,"id":1,"name":"James陈"}}
        Map map2 = JsonUtil.json2map(personJson);
System.out.println(map2.get("key1"));
//{"gender":1,"children":[{"gender":1,"name":"James陈 儿子","id":2,"age":2}],"name":"James陈","id":1,"age":23}
    }

}
```

