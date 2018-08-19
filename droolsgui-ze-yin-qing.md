## **一、Drools简介**

```
Drools 是用Java语言编写的开放源码规则引擎，使用Rete算法对所编写的规则求值。Drools允许使用声明方式表达业务逻辑。可以使用非XML的本地语言编写规则，从而便于学习和理解。并且，还可以将Java代码直接嵌入到规则文件中。
```

Drools相关概念：

* 事实（Fact）：对象之间及对象属性之间的关系。
* 规则（rule）：是由条件和结论构成的推理语句，一般表示为if...Then。一个规则的if部分称为LHS，then部分称为RHS。
* 模式（module）：就是指IF语句的条件。这里IF条件可能是有几个更小的条件组成的大条件。模式就是指的不能在继续分割下去的最小的原子条件。

## **二、Drools简单实例**

1、在pom.xml文件中添加相关的依赖包。

```
<parent>
       <groupId>org.drools</groupId>
       <artifactId>drools</artifactId>
       <version>7.10.0-SNAPSHOT</version>
</parent>
<dependencies>
        <!-- Internal dependencies -->
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-compiler</artifactId>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-decisiontables</artifactId>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-templates</artifactId>
        </dependency>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-internal</artifactId>
        </dependency>
</dependencies>
```

2、创建一个Person实体类

```
package org.drools.examples.test;

public class Person {
    private String name;
    private int age;
    private String desc;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    public String toString() {
        return "[name=" + name + ",age=" + age + ",desc=" + desc + "]";
    }
}
```

3、在 resources 目录下创建一个package存放创建的规则文件PersonRule.drl，内容如下：

```
package com.example.person;

import org.drools.examples.test.Person;
import org.drools.examples.test.DateTimeTest;

global java.util.List list;
global java.util.Date minDate;
global java.util.Date maxDate;

    function void printName(String name,String desc) {
            System.out.println("name:"+name+" desc:"+ desc);
        }

        rule "boy"
            salience 1
            when
                $p : Person(age > 0);
            then
                $p.setDesc("少年");
                retract($p);
                printName($p.getName(),$p.getDesc());
        end

        rule "youth"
            salience 2
            when
                $p : Person(age > 12);
            then
                $p.setDesc("青年");
                retract($p);
                printName($p.getName(),$p.getDesc());
        end

        rule "midlife"
            salience 3
            when
                $p : Person(age > 24);
            then
                $p.setDesc("中年");
                retract($p);
                printName($p.getName(),$p.getDesc());
        end

        rule "old"
        lock-on-active true

          salience 5
            when
                $p : Person(age > 60 && age <80)

            then
                $p.setDesc("老年");
                update($p);
                printName($p.getName(),$p.getDesc());
        end

        rule "containstTest"
            salience 6
            when
                $p : Person(name contains "abcd");
            then
                System.out.println($p.getName()+"包含字符串：abcd");
        end

        rule "memberofTest"
            salience 7
            when
                $p : Person(name memberOf list);
            then
                System.out.println($p.getName()+"存在list中");
        end

       rule "dateTest"
            salience 8
            when
                $d : DateTimeTest(date > minDate && date < maxDate);
            then
                System.out.println($d.getDate()+"在日期范围内");
        end

       rule "dateEffectiveTest"
             salience 8
             date-effective "9-八月-2018"
             date-expires "11-八月-2018"
             when
                 $d : DateTimeTest();
             then
                 System.out.println($d.getDate()+"在日期生效范围内");
        end

      rule "text"
            salience 9
            enabled false
            lock-on-active true
            when
                $p : Person(name contains "abcd" && name memberOf list);
            then
                $p.setName("小明");
                update($p);
                System.out.println($p.getName()+"包含字符串：abcd,并且在list中");
       end

      rule "rule2"
            activation-group "test"
            salience 10
            when
                 eval(true)
            then
                 System.out.println("rule2 execute");
      end

      rule "rule1"
            activation-group "test"
            salience 9
            when
                 eval(true)
            then
                 System.out.println("rule1 execute");
      end
```

4、在 src/main/resources/META-INF/ 目录下创建 kmodule.xml 文件，kmodule.xml 用来描述知识库资源的选择及知识库与会话的配置，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://www.drools.org/xsd/kmodule">
    <kbase name="simpleRuleTest" packages="org.drools.examples.test">
        <ksession name="simpleRuleKSession"/>
    </kbase>
</kmodule>
```

（1）Kmodule 中可以包含一个到多个 kbase,分别对应 drl 的规则文件，kbase的name可取任意字符串但不能重名；

（2）packages：对应src/main/resources/下存放规则文件的文件夹名称，可定义多个包，用逗号隔开。默认情况下会扫描 resources目录下所有\(包含子目录\)规则文件；

（3）kbase里可有一个或多个ksession，每个ksession的name必须设置并且唯一。

5、创建测试类PersonTest，内容如下：

```
package org.drools.examples.test;

import org.kie.api.KieServices;
import org.kie.api.runtime.KieContainer;
import org.kie.api.runtime.KieSession;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class PersonTest {
    static KieSession getSession() {
        KieServices ks = KieServices.Factory.get();
        KieContainer kc = ks.getKieClasspathContainer();
        return kc.newKieSession("simpleRuleKSession");
    }


    public static void run() {
        KieSession ks = getSession();

        List<String> list = new ArrayList<String>();
        list.add("abcdfdjh");
        list.add("佟湘玉");
        list.add("祝无双");
        ks.setGlobal("list",list);

        DateTimeTest data = new DateTimeTest();
        DateFormat df = new SimpleDateFormat("dd-MM-yyyy HH:mm:ss");
//        System.setProperty("drools.dateformat","dd-MM-yyyy HH:mm:ss");
        String s = df.format(new Date());
        try {
            Date minDate = df.parse("10-08-2018 18:00:00");
            Date maxDate = df.parse("10-08-2018 19:00:00");
            ks.setGlobal("minDate",minDate);
            ks.setGlobal("maxDate",maxDate);
            data.setDate(df.parse(s));
        }catch (Exception e){
            e.printStackTrace();
        }
        ks.insert(data);

        Person p1 = new Person("abcdfdjh", 68);
        Person p2 = new Person("abcd123456", 32);
        Person p3 = new Person("佟湘玉", 18);
        Person p4 = new Person("郭芙蓉", 8);
        Person p5 = new Person("祝无双", 66);

        System.out.println("before p1 : " + p1);
        System.out.println("before p2 : " + p2);
        System.out.println("before p3 : " + p3);
        System.out.println("before p4 : " + p4);
        ks.insert(p1);
        ks.insert(p2);
        ks.insert(p3);
        ks.insert(p4);
        ks.insert(p5);

        int count = ks.fireAllRules();
        System.out.println("p1="+p1);
        System.out.println("总执行了" + count + "条规则------------------------------");
        System.out.println("after p1 : " + p1);
        System.out.println("after p2 : " + p2);
        System.out.println("after p3 : " + p3);
        System.out.println("after p4 : " + p4);
        System.out.println("after p5 : " + p5);
        ks.dispose();
    }

    public static void main(String[] args) {
        run();
    }
}
```

代码解释：首先通过请求获取 KieServices，通过KieServices获取KieContainer，KieContainer加载规则文件并获取KieSession，KieSession来执行规则引擎，KieSession是一个轻量级组建，每次执行完销毁。

6、运行结果如下：

![](/assets/1533900757%281%29.png)

## **三、Drools规则语法**

1、规则文件

一个标准的规则文件的结构代码包含如下部分：

（1）package package-name\(包名，必须的，不必和物理路径一致\)，package在规则文件放在第一行，其他的可以无序

（2）import 需要导入Java Bean的完整路径

（3）globals 变量类型 变量名称\(全局变量\)

（4）functions \(函数\)

（5）queries \(查询\)（暂未理解清楚）

（6）rules 规则名称，不可重名

2、规则语言

```
package 包名

rule "规则名"
when
    (条件) - 也叫作规则的 LHS(Left Hand Side)
then
    (动作/结果) - 也叫作规则的 RHS(Right Hand Side)
end
```

（1）绑定参数语法：$\[绑定变量名\] : Object\(\[field 约束\]\)

例如：$p: Person\(\) //p绑定外面传入的Person对象

（2）条件判断：$绑定变量名 ：绑定类型\(属性1 比较符合 比较值\)

例如：$p: Person\(age &gt; 60 && age &lt; 80\) //必须满足age &gt; 60 和age &lt; 80的Person

（3）约束连接：对于对象内部的多个约束的连接，可以采用“&&”（and）、“\|\|”\(or\)和“,”\(and\)来实现。“，”与“&&”和“\|\|”不能混合使用，也就是说在有“&&”或“\|\|”出现的 LHS 当中，是不可以有“，”连接符出现的，反之亦然。

（4）比较操作符：十二种类型的比较操作符，分别是：&gt;、&gt;=、&lt;、&lt;=、= =、!=、contains、not contains、memberof、not memberof、matches、not matches。

contains：是用来检查一个 Fact 对象的某个字段（该字段要是一个 Collection或是一个 Array 类型的对象）是否包含一个指定的对象。

memberof：是用来判断某个 Fact 对象的某个字段是否在一个集合（Collection/Array）当中，用法与 contains 有些类似，但也有不同。

matches：是用来对某个 Fact 的字段与标准的 Java 正则表达式进行相似匹配，被比较的字符串可以是一个标准的 Java 正则表达式，但有一点需要注意，那就是正则表达式字符串当中不用考虑“\”的转义问题。

3、常用语法属性

（1）dialect：设置规则当中要使用的语言类型，目前支持：mvel和Java，默认是Java。

mvel语法：

表示对象的属性 ：user.name   相当于java代码 user.getName\(\)

```
               user.manager.name相当于Java代码user.getManager\(\).getName\(\)
```

（2）salience ：设置规则执行的优先级，salience 属性的值是一个数字，数字越大执行优先级越高,，同时它的值可以是一个负数。默认情况下，规则的 salience 默认值为 0。如果不设置规则的 salience 属性，那么执行顺序是随机的。

（3）no-loop ：定义当前的规则是否不允许多次循环执行，,默认是 false，也就是当前的规则只要满足条件，可以无限次执行。

（4）lock-on-active ：将lock-on-active属性的值设置为true，可避免因某些Fact对象被修改而使已经执行过的规则再次被激活执行。

（5）date-effective：设置规则的生效时间，只有date-effective设置的时间值&lt;=当系统时间时，规则才会触发执行，否则执行将不执行。date-effective 可接受的日期格式为“dd-MMM-yyyy”。

（6）date-expires：设置规则的过期时间，只有date-expires设置的时间值&gt;=当系统时间时，规则才会触发执行，否则执行将不执行。

（7）enabled：设置规则是否可用。

（8）activation-group：将若干个规则分成一个组，用一个字符串来给这个组命名，这样在执行的时候，具有相同 activation-group 属性的规则中只要有一个会被执行，其它的规则都将不再执行。

（9）agenda-group：引擎在调用这些设置了 agenda-group 属性的规则的时候需要显示的指定某个 Agenda Group 得到 Focus（焦点），这样位于该 Agenda Group 当中的规则才会触发执行，否则将不执行。

（10）auto-focus：作用是用来在已设置了 agenda-group 的规则上设置该规则是否可以自动独取 Focus，如果该属性设置为 true，那么在引擎执行时，就不需要显示的为某个 Agenda Group 设置 Focus，否则需要。

（11）ruleflow-group：用来将规则划分为一个个的组，然后在规则流当中通过使用 ruleflow-group 属性的值，从而使用对应的规则。

4、常用函数

（1）insert\(Object o\)：实现对当前Working Memory中的Fact 对象进行新增。

（2）insertLogical\(Object o\)：与insert类似，但是当没有更多事实支持当前触发规则的事实时，对象将被自动删除。

（3）update\(Object o\)：实现对当前Working Memory中的Fact 对象进行更新。

（4）modify\(Object o\)：实现对当前Working Memory中的Fact 对象进行更新，与update语法不同，结果都是更新操作。

（5）retract\(Object o\)：将Working Memory 当中Fact 对象从Working Memory 当中删除。

5、常用方法

（1）求和：Long\(...\) from accumulate\(..., sum\($p\)\)

```
例如：$i : Double(doubleValue > 100) from accumulate (
                Purchase( customer == $c, $price : product.price ),
                sum( $price )
        )
```



