---
title: " Java对象和Xml的互相转换\t\t"
tags:
  - xml
url: 140.html
id: 140
comments: false
categories:
  - 杂
date: 2018-10-19 10:20:16
---
```java
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlElementWrapper;
import javax.xml.bind.annotation.XmlRootElement;
```
`@XmlRootElement(name = "ROOT")`定义根元素的名字
`@XmlElement(name = "NAME")`定义根元素下面元素的名字，一般写在getter/setter上
`@XmlElementWrapper(name = "HABITLIST")`可以遍历ListT和Set，但是对于map的支持不是很好
实体类
```java
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlElementWrapper;
import javax.xml.bind.annotation.XmlRootElement;
import java.util.List;


@XmlRootElement(name = "ROOT")
public class XmlBean {
    private String name ;

    private int age;

    private List<Habit> list;

    @XmlElement(name = "NAME")
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @XmlElement(name = "AGE")
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @XmlElementWrapper(name = "HABITLIST")
    @XmlElement(name = "HABIT")
    public List<Habit> getList() {
        return list;
    }

    public void setList(List<Habit> list) {
        this.list = list;
    }
}
---------------------------------------------------------------------------------
public class Habit {

    private String aboutLife;
    private String aboutLern;

    @XmlElement(name = "ABOUTLEARN")
    public String getAboutLern() {
        return aboutLern;
    }

    public void setAboutLern(String aboutLern) {
        this.aboutLern = aboutLern;
    }

    public String getAboutLife() {
        return aboutLife;
    }

    @XmlElement(name = "ABOUTLIFE")
    public void setAboutLife(String aboutLife) {
        this.aboutLife = aboutLife;
    }
}
```
测试代码
```java
    @Test
    public void test1(){
        XmlBean xmlBean = new XmlBean();
        xmlBean.setAge(15);
        xmlBean.setName("张三");

        Habit habit = new Habit();
        habit.setAboutLern("学习");
        habit.setAboutLife("生活");

        Habit habit1 = new Habit();
        habit1.setAboutLern("学习1");
        habit1.setAboutLife("生活1");

        List<Habit> list = new ArrayList<>();
        list.add(habit);
        list.add(habit1);

        xmlBean.setList(list);
        try {
            String s = beanToXml(xmlBean, XmlBean.class);
            System.out.println(s);
        } catch (JAXBException e) {
            e.printStackTrace();
        }
    }
```
生成的结果
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ROOT>
    <AGE>15</AGE>
    <HABITLIST>
        <HABIT>
            <ABOUTLEARN>学习</ABOUTLEARN>
            <ABOUTLIFE>生活</ABOUTLIFE>
        </HABIT>
        <HABIT>
            <ABOUTLEARN>学习1</ABOUTLEARN>
            <ABOUTLIFE>生活1</ABOUTLIFE>
        </HABIT>
    </HABITLIST>
    <NAME>张三</NAME>
</ROOT>
```
xml和Java对象的互相转换
```java
public static String beanToXml(Object obj,Class<?> load) throws JAXBException{
        JAXBContext context = JAXBContext.newInstance(load);
        Marshaller marshaller = context.createMarshaller();
        marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);//格式化xml
        marshaller.setProperty(Marshaller.JAXB_ENCODING, "UTF-8");
        StringWriter writer = new StringWriter();
        marshaller.marshal(obj,writer);
        return writer.toString();
    }

    public static Object xmlToBean(String xml,Class<?> load) throws JAXBException, IOException {
        JAXBContext context = JAXBContext.newInstance(load);
        Unmarshaller unmarshaller = context.createUnmarshaller();
        Object unmarshal = unmarshaller.unmarshal(new StringReader(xml));
        return unmarshal;
    }
```