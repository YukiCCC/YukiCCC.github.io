---
title: Java-集合
date: 2023-06-29 22:07:43
tags: Java
category: Java
---
## 1. Map

### 集合框架类图

![image-20220424185652481](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424185652481.png)

### HashMap

#### 常用方法

![image-20220424190445414](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424190445414.png)
键值对（“key = value”），顾名思义，每一个键会对应一个值。

![image-20220424190535142](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424190535142.png)

#### API

API 是用于构建应用程序软件的一组子程序定义，协议和工具。一般来说，这是一套明确定义的各种软件组件之间的通信方法。

![image-20220424190638466](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424190638466.png)

#### 例1：

```java
/*1.遍历集合，并将序号与对应人名打印。
2.向该map插入一个编码为5姓名为李晓红
3.移除该map中的编号为1的信息
4.将map集合中编号为2的姓名信息修改为"周琳"*/

import java.util.HashMap;
import java.util.Map;

public class Ex01 {
    public static void main(String[] args) {
        Map map =new HashMap();//多态
        map.put(1,"张三丰");
        map.put(2,"周芷若");
        map.put(3,"汪峰");
        map.put(4,"灭绝师太");
        //1.循环遍历
        map.forEach((k,v)-> System.out.println("key:"+k+"，value:"+v));
        //2.插入
        map.put(5,"李晓红");
        //3.移除
        map.remove(1);
        //4.修改
        //map.put(2,"周琳");
        map.replace(2,"周琳");
        System.out.println("==============================");
        map.forEach((k,v)-> System.out.println("key:"+k+"，value:"+v));

    }
}
```

java8 参考循环输出：

```java
Map map = new HashMap() ;
map.put(1,"张三1") ;
map.put(2,"张三2") ;
map.put(3,"张三3") ;
// java lambda表达式  类似 ES6 的箭头函数 【可推导即可省略】
map.forEach((k,v)-> System.out.println("序号:"+k+",姓名:"+v));
```



#### 例2：

```java
/*有2个数组，
第一个省份数组内容为：
[黑龙江省,浙江省,江西省,广东省,福建省]，
第二个省会数组为：
[哈尔滨,杭州,南昌,广州,福州]，
将第一个数组元素作为key，第二个数组元素作为value存储到Map集合中。
如{黑龙江省=哈尔滨, 浙江省=杭州, …}。*/
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class Ex02 {
    public static void main(String[] args) {
        String[] proAy = {"黑龙江省","浙江省","江西省","广东省","福建省"} ;
        String[] cityAy = {"哈尔滨","杭州","南昌","广州","福州"} ;

        Map map = new LinkedHashMap();
        // 把 数组中的元素  存放在 map的 条目中
        for (int i=0;i<proAy.length;i++){
            map.put(proAy[i],cityAy[i]);
        }

        //打印map  无需循环  {key=value,key=value}
        System.out.println(map);
    }
}
```



#### 例3：

```java
/*定义一个泛型为String类型的List集合，统计该集合中每个字符（注意，不是字符串）出现的次数。 
	List list = new ArrayList() ;
例如：集合中有”abc”、”bcd”两个元素，
     list.add(“abc”)  ;    
     list.add(“bcd”) ;
程序最终输出结果为：“a = 1,b = 2,c = 2,d = 1”。    
  Map map = ...
   String 类  1.  length() 字母个数
              2.   charAt() 根据索引 获得 字母*/


```



### Hashtable

```
null值问题?
```

### 对比：

![image-20220424191449662](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424191449662.png)

## 2. Iterator

### 基本使用

![image-20220424191752185](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424191752185.png)

![image-20220424191841510](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424191841510.png)

#### 例4

```java
/*使用ArrayList存储多个学生信息
1. 删除年龄>18岁的学生
2. 使用Iterator进行遍历 */
import java.util.ArrayList;
import java.util.Iterator;
import java.util.function.Predicate;

/**
 * 使用ArrayList存储多个学生信息
 * 1. 删除年龄>18岁的学生
 * 2. 使用Iterator进行遍历
 */
public class Ex04 {
    public static void main(String[] args) {
        ArrayList<Stu> stuList = new ArrayList();
        stuList.add(new Stu(19)) ;
        stuList.add(new Stu(39)) ;
        stuList.add(new Stu(14)) ;
        stuList.add(new Stu(16)) ;
        stuList.add(new Stu(26)) ;
        stuList.add(new Stu(12)) ;

        //推荐 java8的语法   removeIf(e->e.getAge()>18)   forEach(System.out::println)
        //stuList.removeIf( o -> ((Stu)o).getAge()>18 ) ;

        //stuList.removeIf( o -> o.getAge()>18 ) ;
        //stuList.forEach(System.out::println);


        //Iterator 判断有没有下一个元素  hasNext()    取出下一个元素  next()   删除当前元素 remove()
        Iterator<Stu> iterator = stuList.iterator();
        while (iterator.hasNext()) {
            //腐烂味道代码   坏味道代码
             Stu stu = iterator.next();
            if (stu.getAge()>18){
                iterator.remove();
            }
        }

        System.out.println(stuList);
    }
}


```



参考java8 List 删除元素：

```java
List list = new ArrayList() ;
list.add(new Stu(1,"aaa",19)) ;
list.add(new Stu(2,"bbb",16)) ;
list.add(new Stu(3,"ccc",14)) ;
//如果满足条件  进行删除
list.removeIf(ele->((Stu)ele).getAge()>18) ;
//循环输出
list.forEach(System.out::println);
```



## 3. 泛型

### 为什么

![image-20220424192006124](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424192006124.png)

### 怎么用

![image-20220424192037339](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424192037339.png)

![image-20220424192106503](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424192106503.png)

## 4. Set

### 4.1 HashSet

![image-20220424192144146](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220424192144146.png)

#### 例5

```java
import java.util.HashSet;
import java.util.Random;
import java.util.Set;

/**
 * 双色球规则：
 * 双色球每注投注号码由6个红色球号码和1个蓝色球号码组成。
 * 红色球号码从1—33中选择；
 * 蓝色球号码从1—16中选择；
 * 请随机生成一注双色球号码。
 * （要求同色号码不重复）
 */
public class Ex05 {
    public static void main(String[] args) {
        Random random = new Random();
        // 蓝色球号码
        int blueBall = random.nextInt(16)+1 ;

        HashSet<Ball> ballSet = new LinkedHashSet<>() ;
        // 一个 蓝色球
        ballSet.add(new Ball("蓝球",blueBall)) ;

        while (ballSet.size() != 7) {
            //红色球号码
            int redBall = random.nextInt(33)+1 ;
            ballSet.add(new Ball("红球",redBall)) ;
        }


        System.out.println(ballSet);
    }
}


class Ball{
    private String color ;
    private Integer num ;

    public Ball(String color, Integer num) {
        this.color = color;
        this.num = num;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public Integer getNum() {
        return num;
    }

    public void setNum(Integer num) {
        this.num = num;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Ball ball = (Ball) o;
        return Objects.equals(color, ball.color) &&
                Objects.equals(num, ball.num);
    }

    @Override
    public int hashCode() {
        return Objects.hash(color, num);
    }

    @Override
    public String toString() {
        return "Ball{" +
                "color='" + color + '\'' +
                ", num=" + num +
                '}';
    }
}

```
## 5. Map练习

### 5.1 Map练习

#### 例6

```java
一、利用Map，完成下面的功能：
/*从命令行读入一个字符串，表示一个年份，输出该年的世界杯冠军是哪支球队。
如果该年没有举办世界杯，则输出：没有举办世界杯。*/
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Ex06 {
    public static void main(String[] args) {
            Map<Integer, String> map = new HashMap<>();
            map.put(1930, "乌拉圭");
            map.put(1934, "意大利");
            map.put(1938, "意大利");
            map.put(1950, "乌拉圭");
            map.put(1954, "西德");
            map.put(1958, "巴西");
            map.put(1962, "巴西");
            map.put(1966, "英格兰");
            map.put(1970, "巴西");
            map.put(1974, "西德");
            map.put(1978, "阿根廷");
            map.put(1982, "意大利");
            map.put(1986, "阿根廷");
            map.put(1990, "西德");
            map.put(1994, "巴西");
            map.put(1998, "法国");
            map.put(2002, "巴西");
            map.put(2006, "意大利");
            map.put(2010, "西班牙");
            map.put(2014, "德国");
            //map.put(2023,null);

            //键盘使用工具类
            Scanner scanner = new Scanner(System.in);
            System.out.println("请输入年份:");
            //怼死  死去活来法
            int year = scanner.nextInt();
            String msg = map.getOrDefault(year, "没有举办世界杯");
            System.out.println(msg);



        scanner.close();

        /*String team = map.get(year);
        if (team!=null){
            System.out.println(team);
        }else{
            System.out.println("没有举办世界杯");
        }*/

        /*if (map.containsKey(year)) {
            String team = map.get(year);
            System.out.println(team);
        }else{
            System.out.println("没有举办世界杯");
        }*/
       /* String msg = map.getOrDefault(year, "没有举办世界杯");
        System.out.println(msg);*/
    }
}

```

#### 例7

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Scanner;

/**
 * 二、在原有世界杯Map 的基础上，
 *     增加如下功能： 
 *     	读入一支球队的名字，输出该球队夺冠的年份列表。 
 *     例如，
 *         读入“巴西”，应当输出 1958 1962 1970 1994 2002 
 *         读入“荷兰”，应当输出 没有获得过世界杯 
 */
public class Ex07 {
    public static void main(String[] args) {
            Map<Integer, String> map = new HashMap<>();
            map.put(1930, "乌拉圭");
            map.put(1934, "意大利");
            map.put(1938, "意大利");
            map.put(1950, "乌拉圭");
            map.put(1954, "西德");
            map.put(1958, "巴西");
            map.put(1962, "巴西");
            map.put(1966, "英格兰");
            map.put(1970, "巴西");
            map.put(1974, "西德");
            map.put(1978, "阿根廷");
            map.put(1982, "意大利");
            map.put(1986, "阿根廷");
            map.put(1990, "西德");
            map.put(1994, "巴西");
            map.put(1998, "法国");
            map.put(2002, "巴西");
            map.put(2006, "意大利");
            map.put(2010, "西班牙");
            map.put(2014, "德国");

            //逆向思维
            Map<String, StringBuilder> teamMap = new HashMap<>() ;
            map.forEach((year,name)->{
                    //当前 队伍 map集合中 是否包含当前 队伍
                    if (teamMap.containsKey(name)) {
                            // 字符串 缓存中 添加新的年份
                            StringBuilder sb = teamMap.get(name);
                            sb.append(year).append("\t") ;
                    }else {
                            //如果不包含  直接存放当前年份
                            teamMap.put(name, new StringBuilder(year + "\t"));
                    }
            });

           // System.out.println(teamMap);
            //键盘使用工具类
            Scanner scanner = new Scanner(System.in);
            System.out.println("请输入一支球队的名字:");
            //怼死  死去活来法
            String year = scanner.next();
            String msg = teamMap.getOrDefault(year, new StringBuilder("没有获得过世界杯")).toString();
            System.out.println(msg);
    }
}
```
```java
//另一种方法
            String inputTeamName = scanner.next();
            AtomicBoolean isPrint = new AtomicBoolean(true);
            map.forEach((year,name)->{
                   if (name.equals(inputTeamName)){
                           System.out.println(year);
                           //System.exit(-1);
                           isPrint.set(false);
                   }
            });


            if (isPrint.get()) {
                    System.out.println("没有获得过世界杯");
            }
```
```java
//第三种方法
            String inputTeamName = scanner.next();

            //存放 获得世界杯的年份集合
            List<Integer> yearList = new ArrayList<>() ;
            map.forEach((year,name)->{
                   if (name.equals(inputTeamName)){
                           yearList.add(year) ;
                   }
            });


            //集合为空  条件没有满足
            if (yearList.isEmpty()) {
                    System.out.println("没有获得过世界杯");
            }

            yearList.forEach(System.out::println);
```

### 5.2 Map.Entry

![image-20220425211411275](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425211411275.png)

### 5.3 Map综合案例

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * 站编号和站名对应关系如下：
 * 1=朱辛庄
 * 2=育知路
 * 3=平西府
 * 4=回龙观东大街
 * 5=霍营
 * 6=育新
 * 7=西小口
 * 8=永泰庄
 * 9=林萃桥
 * 10=森林公园南门
 * 11=奥林匹克公园
 * 12=奥体中心
 * 13=北土城
 *
 * 将以上对应关系的数据存储到map集合中，key：表示站编号，value：表示站名，并遍历打印(可以不按顺序打印)：
 *     例如：
 *        第10站: 森林公园南门
 *        第6站: 育新
 *        第12站: 奥体中心
 *        第13站: 北土城
 */
public class EX {
    public static void main(String[] args) {
        String[] nameAy = {"朱辛庄","育知路","平西府","回龙观东大街","霍营","育新","西小口","永泰庄","林萃桥","森林公园南门","奥林匹克公园","奥体中心","北土城"} ;
//        Map<Integer,String> map = new HashMap<>(nameAy.length) ;
        Map<Integer,String> map = new LinkedHashMap<>(nameAy.length) ;
        for (int i = 0; i < nameAy.length; i++) {
            map.put(i+1,nameAy[i]) ;
        }

        //打印 map
        map.forEach((idx,name)-> System.out.println("第"+idx+"站:"+name));
    }
}

```

#### 例8

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * 站编号和站名对应关系如下：
 * 1=朱辛庄
 * 2=育知路
 * 3=平西府
 * 4=回龙观东大街
 * 5=霍营
 * 6=育新
 * 7=西小口
 * 8=永泰庄
 * 9=林萃桥
 * 10=森林公园南门
 * 11=奥林匹克公园
 * 12=奥体中心
 * 13=北土城
 *
 *计算地铁票价规则：
 *     总行程 3站内（包含3站）收费3元，
 *     3站以上但不超过5站（包含5站）的收费4元，
 *     5站以上的，在4元的基础上，每多1站增加2元，
 *     10元封顶；
 */
public class Ex08 {
    public static void main(String[] args) {
        String[] nameAy = {"朱辛庄","育知路","平西府","回龙观东大街","霍营","育新","西小口","永泰庄","林萃桥","森林公园南门","奥林匹克公园","奥体中心","北土城"} ;
//        Map<Integer,String> map = new HashMap<>(nameAy.length) ;
        Map<Integer,String> map = new LinkedHashMap<>(nameAy.length) ;
        for (int i = 0; i < nameAy.length; i++) {
            map.put(i+1,nameAy[i]) ;
        }

        //打印 map
        map.forEach((idx,name)-> System.out.println("第"+idx+"站:"+name));

        //站之间的数量
        int money = caclMoney(13) ;
        System.out.println("共:" + money);
    }


    /**
     * 计算地铁票价规则：
     *      总行程 3站内（包含3站）收费3元，
     *    3站以上但不超过5站（包含5站）的收费4元，
     *       5站以上的，在4元的基础上，每多1站增加2元，
     *     10元封顶；
     * @param count  站之间的数量
     * @return  多少钱
     */
    private static int caclMoney(int count){
        Map<Integer,Integer> moneyMap = new HashMap<>() ;
        moneyMap.put(0,3) ;
        moneyMap.put(1,3) ;
        moneyMap.put(2,3) ;
        moneyMap.put(3,3) ;
        moneyMap.put(4,4) ;
        moneyMap.put(5,4) ;
        // ....
        moneyMap.put(0,3) ;
        moneyMap.put(0,3) ;

        return  count ;
        /*if (count<=3){
            return 3 ;
        }
        if (count<=5){
            return 4 ;
        }
        int money = 4+(count-5)*2 ;
        if (money>10){
            return 10 ;
        }
        return money ;*/
    }
}

```

#### 例9

```java
/*计算地铁票价规则：
    总行程 3站内（包含3站）收费3元，
    3站以上但不超过5站（包含5站）的收费4元，
    5站以上的，在4元的基础上，每多1站增加2元，
    10元封顶；
*/
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

public class Test09 {
    public static void main(String[] args) {
        ArrayList<Num> numList = new ArrayList();
        Map map = new HashMap();
        map.put(1,"朱辛庄");
        map.put(2,"育知路");
        map.put(3,"平西府");
        map.put(4,"回龙观东大街");
        map.put(5,"霍营");
        map.put(6,"育新");
        map.put(7,"西小口");
        map.put(8,"永泰庄");
        map.put(9,"林萃桥");
        map.put(10,"森林公园南门");
        map.put(11,"奥林匹克公园");
        map.put(12,"奥体中心");
        map.put(13,"北土城");
        for (Object j : map.keySet()){

            System.out.println("第" +j +"站: " +map.get(j));
        }
        int i=1;
        numList.add(new Num(1,3)) ;
        numList.add(new Num(2,3)) ;
        numList.add(new Num(3,3)) ;
        numList.add(new Num(4,4)) ;
        numList.add(new Num(5,4)) ;
        numList.add(new Num(5+i,4+2*i)) ;
        /*numList.forEach((k,v)->{
            if(k.equals()<=3){
                System.out.println(k + " ");
            }
        });*/
    }
}
class Num{
    int number ;
    int money;

    public int getMoney() {
        return money;
    }

    public int getNumber() {
        return number;
    }

    public Num(int number,int money) {
        this.number = number;
        this.money = money;
    }

    @Override
    public String toString() {
        return "homework.Num{" +
                "number=" + number +
                ", money=" + money +
                '}';
    }
}


```

#### 例10

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * 站编号和站名对应关系如下：
 * 1=朱辛庄
 * 2=育知路
 * 3=平西府
 * 4=回龙观东大街
 * 5=霍营
 * 6=育新
 * 7=西小口
 * 8=永泰庄
 * 9=林萃桥
 * 10=森林公园南门
 * 11=奥林匹克公园
 * 12=奥体中心
 * 13=北土城
 *
 打印格式（需要对键盘录入的上车站和到达站进行判断，如果没有该站，提示重新输入，直到站名存在为止）：
 注意：每站需要2分钟
 请输入上车站：
 朱辛庄
 请输入到达站：
 西小口
 从朱辛庄到西小口共经过6站收费6元，大约需要 12分钟；
 */
public class Ex09 {
    private  static String[] nameAy = {"朱辛庄","育知路","平西府","回龙观东大街","霍营","育新","西小口","永泰庄","林萃桥","森林公园南门","奥林匹克公园","奥体中心","北土城"} ;
    private  static Map<Integer,String> map = new LinkedHashMap<>(nameAy.length) ;

    public static void main(String[] args) {
//        Map<Integer,String> map = new HashMap<>(nameAy.length) ;
        for (int i = 0; i < nameAy.length; i++) {
            map.put(i+1,nameAy[i]) ;
        }

        //打印 map
        map.forEach((idx,name)-> System.out.println("第"+idx+"站:"+name));

        String beginName = "林萃桥" ;
        String endName = "平西府";
        int count = caclCount(beginName, endName);
        int time = count*2 ;
        //站之间的数量
        int money = caclMoney(count) ;
        System.out.println("从"+beginName+"到"+endName+"共经过"+count+"站收费"+money+"元，大约需要 "+time+"分钟");
    }

    /**
     * 根据 两站 站名  计算 之间   站之间的数量
     * @param beginName
     * @param endName
     * @return
     */
    private static int caclCount(String beginName,String endName){
        //map的 key站名 value 站序号 反转
        Map<String,Integer> name2IndxMap = new HashMap<>() ;
        //站序号 站名
        map.forEach((idx,name)-> name2IndxMap.put(name,idx));
        Integer beginIdx = name2IndxMap.get(beginName);
        Integer endIdx = name2IndxMap.get(endName);
        return Math.abs(beginIdx-endIdx) ;
    }


    /**
     * 计算地铁票价规则：
     *      总行程 3站内（包含3站）收费3元，
     *    3站以上但不超过5站（包含5站）的收费4元，
     *       5站以上的，在4元的基础上，每多1站增加2元，
     *     10元封顶；
     * @param count  站之间的数量
     * @return  多少钱
     */
    private static int caclMoney(int count){
        if (count<=3){
            return 3 ;
        }
        if (count<=5){
            return 4 ;
        }
        int money = 4+(count-5)*2 ;
        if (money>10){
            return 10 ;
        }
        return money ;
    }
}

```

## 6.Set扩展

### 6.1 TreeSet

![image-20220425211927973](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425211927973.png)

### 6.2 例11

```java
/*学生类  姓名 分数 double
  存放 TreeSet中
  实现 根据 分数 从大到小  
    内部排序Comparable 
    外部排序 Comparator
Set<Stu>  set = new TreeSet<>() ;
set.add(new Stu(“aa”,98.5)) ;
set.add(new Stu(“bb”,88.5)) ;*/
package set;

import java.util.*;

public class Ex11 {
    public static void main(String[] args) {
        //Set<set.Customer> set = new TreeSet<>((c1,c2)-> (int) (c1.getScore()-c2.getScore())) ;
        //Set<set.Customer> set = new TreeSet<>((c1,c2)-> c1.getScore().compareTo(c2.getScore())) ;
//        Set<Customer> set = new TreeSet<>(Comparator.comparing(Customer::getScore)) ;
        //Set<set.Customer> set = new TreeSet<>(Comparator.comparingInt(set.Customer::getScore)) ;
        Set<Customer> set = new TreeSet<>() ;
        set.add(new Customer("小丽",500D)) ;
        set.add(new Customer("小红",400.8)) ;
        set.add(new Customer("小夏",400.3)) ;
        set.add(new Customer("小花",700D)) ;

        set.forEach(System.out::println);
    }
}
```

java8 外部排序写法：

```java
Set<Stu> set = new TreeSet<>(Comparator.comparingInt(Stu::getAge)) ;
```



## 7. Collections

### 7.1 辅助类

![image-20220425212138509](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425212138509.png)

### 7.2 例12

```java
/*  1. List<Stu>  
      Collections.sort(stuList)     Comparable
      Collections.sort(stuList,...)  Comparator
      
分别用Comparable和Comparator两个接口对下列四位同学的成绩做降序排序，
    如果成绩一样，那在成绩排序的基础上按照年龄由小到大排序。*/
package collections;

import set.Customer;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Ex12 {
    public static void main(String[] args) {
        //集合中 存放 Customer
        List<Customer> customerList = Arrays.asList(new Customer("aaa",100D),new Customer("bbb",80.5D),new Customer("ccc",90D)) ;

        //jdk1.8 新特性 不使用辅助类 Collections
        //customerList.sort((c1,c2)->c1.getScore().compareTo(c2.getScore()));
        //使用 辅助类 Collections
        //Collections.sort(customerList);
        Collections.sort(customerList,(c1,c2)->c1.getScore().compareTo(c2.getScore()));
        customerList.forEach(System.out::println);
    }
}

```

![image-20220425212249416](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425212249416.png)

## 8. Map扩展

### 8.1 LinkedHashSet

![image-20220425212514027](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425212514027.png)

### 8.2 LinkedHashMap

![image-20220425212553993](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425212553993.png)

### 8.3 ConcurrentHashMap

![image-20220425212739744](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425212739744.png)

### 8.4 Properties

![image-20220425212815266](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425212815266.png)

## 9.泛型扩展

### 9.1 什么是泛型

![image-20220425213010808](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425213010808.png)

![image-20220425213040282](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425213040282.png)

### 9.2 泛型好处

![image-20220425213114472](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425213114472.png)

### 9.3 使用前后对比

![image-20220425213345463](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425213345463.png)

![image-20220425213500007](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425213500007.png)

### 9.4 类型参数

![image-20220425214125508](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214125508.png)

![image-20220425214149193](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214149193.png)

### 9.5 方法参数

![image-20220425214235909](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214235909.png)

![image-20220425214350680](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214350680.png)

![image-20220425214501978](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214501978.png)

![image-20220425214614120](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214614120.png)

### 9.6 泛型不是协变的

![image-20220425214656674](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214656674.png)

### 9.7 类型通配符

![image-20220425214746410](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214746410.png)

![image-20220425214810914](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214810914.png)

### 9.8 泛型局限性

![image-20220425214846267](https://raw.githubusercontent.com/YukiCCC/figure/main/image-20220425214846267.png)
