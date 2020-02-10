# CatMock

CatMock是一个[mock.js](http://mockjs.com/)的Java封装库。使用JDK自带的js脚本引擎直接调用mock.js脚本，实现对mock.js的统一。

## 为什么采用这样的方案？
在开发互联网应用时，前后端在开发前使用接口管理平台设计接口。市面上常见的接口管理平台大部分使用了Mock.js作为虚拟接口和接口测试的工具。

设计接口时，接口的定义是项目的重要资产。Mock.js能很好地完成后端接口的单元测试。但后端在CI/CD进行集成测试时，往往需要自行编写测试用例，需要使用其它方式生成虚拟数据，无法复用接口设计阶段的项目资产。

于是开始尝试使用纯Java编写一个兼容Mock.js语法的Mock工具：为了实现拓展自定义函数的功能，曾设计使用Java的Function接口对象作为用户自定义传入的函数来使用，但最后放弃了这一想法，因为写工具的初衷在于复用接口设计阶段的项目资产，而Function需要后端重新编写。

最后，选择在Java中使用Java内置的JavaScript引擎直接调用mock.js，这样能保证和mock.js统一，保证接口设计阶段的项目资产能被复用。

## Maven
```xml
<dependency>
    <groupId>cn.myzju.mock</groupId>
    <artifactId>CatMock</artifactId>
    <version>1.0.1</version>
</dependency>
```
## 使用说明

### 获取CatMock对象

```java
//使用内置的mock.js文件初始化
CatMock catMock = new CatMock();
//使用外置的mock.js文件进行初始化
CatMock catMock = new CatMock(new FileReader("{path}/mock.js"));
```

### Random

传入参数：`function`或`function('args')`，返回类型为String

```java
catMock.random("string"); //👉P^7
catMock.random("lower(\"HELLO\")");//👉hello
```

**当前已知不兼容函数：** 
- dataImage()

### Mock

示例可参考[mock.js官方示例](http://mockjs.com/examples.html)

**生成Java对象**

传入字符串必须以`{`开始，并以`}`结束

```java
Person person = catMock.mockObject("{\"name\":\"@string\"}",Person.class);
```

**生成Java List**

传入字符串必须以`[`开始，并以`]`结束

```java
List<Person> persons = catMock.mockArray("[{\"name\":\"@string\"},{\"name\":\"@string\"}]",Person.class); 
```

**生成String**

mock方法传入非JSON格式的字符串（如下`e.g.1`所示）时，前后一定要加上单引号。

```java
//e.g.1
catMock.mock("'@name'");//👉Anna Jackson
//e.g.2
catMock.mock("{\n'regexp|1-5': /\\d{5,10}\\-/\n}");//👉{"regexp": "5912165-6588485-0462848-"}
```

### Extend

生成CatMock对象后，可以通过extend加载自定义函数。

例如新增一个名为`constellation`，用于获取随机星座名称的方法，JavaScript代码如下：

```javascript
{
    constellation: function(date) {
        var constellations = ['白羊座', '金牛座', '双子座', '巨蟹座', '狮子座', '处女座', '天秤座', '天蝎座', '射手座', '摩羯座', '水瓶座', '双鱼座']
        return this.pick(constellations)
    }
}
```

在CatMock中，将脚本代码直接以String类型传入extend函数：

```java
CatMock catMock = new CatMock()
catMock.extend("{\n" +
        "    constellation: function(date) {\n" +
        "        var constellations = ['白羊座', '金牛座', '双子座', '巨蟹座', '狮子座', '处女座', '天秤座', '天蝎座', '射手座', '摩羯座', '水瓶座', '双鱼座']\n" +
        "        return this.pick(constellations)\n" + 
        "    }\n" +
        "}");
catMock.random("constellation")//👉水瓶座
catMock.mock("'@constellation'")//👉白羊座
```
## Development Plan

- mock.js后续版本的兼容
- 变量容器：容器内可以存储各种变量，并能通过`xxx.xxx[index].xxx`的key值进行调用容器内的变量

## License

CatMock is available under the terms of the [MIT License](http://opensource.org/licenses/MIT).
