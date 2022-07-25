# java学习
JAVA教程:
[菜鸟教程](https://www.runoob.com/java/java-basic-syntax.html "java基础语法")  
[炼码](https://www.lintcode.com/course/1/learn/?chapterId=1&sectionId=2 "Java 基础语法：语法、变量与运算")
## 基础知识
- JDK ： Java Development ToolKit (Java 开发工具包)。 JDK 是整个 Java 的核心，包括了 Java 运行环境（Java Runtime Environment），一些 Java 工具（javac/java/jdb 等）和 Java 基础的类库（即Java API 包括 ```rt.jar``` ）。
- JRE：Java Runtime Environment (Java 运行时环境)。也就是我们说的 Java 平台，所有的 Java 程序都要在 JRE 下才能运行。包括 JVM 和 Java 核心类库和支持文件。
- JVM：Java Virtual Mechinal (Java 虚拟机)。JVM 是 JRE 的一部分，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的

## 基础语法
类、对象、方法和实例变量的概念:
- **对象**：对象是类的一个实例，有状态和行为。例如，一条狗是一个对象，它的状态- 有：颜色、名字、品种；行为有：摇尾巴、叫、吃等。
- **类**：类是一个模板，它描述一类对象的行为和状态。
- **方法**：方法就是行为，一个类可以有很多方法。逻辑运算、数据修改以及所有动作-都是在方法中完成的。
- **实例变量**：每个对象都有独特的实例变量，对象的状态由这些实例变量的值决定。

*一次尝试*
```
pulic class hello{
     public static void main{   
        System.out.println("hello");
    }
}  
```
>*pulic class hello*  
public表示这个类是公开的
class用来定义一个类
Hello是类的名字，按照习惯，首字母H要大写，{}中间则是类的定义。  

>*public static void main(String[] args){}*
public、static用来修饰方法，这里表示它是一个公开的静态方法
void是方法的返回类型
{}中间的就是方法的代码
方法是可执行的代码块，一个方法包括方法名和方法参数
main是方法名
()括起来的方法参数，这里的main方法有一个参数
String[]是参数类型
args是参数名


*输出展示*
hello

**Java基础语法**
- **大小写敏感**：Java是大小写敏感的，这就意味着标识符Hello与hello是不同的。
- **类名**：对于所有的类来说，类名的首字母应该大写。如果类名由若干单词组成，那么每个单词的首字母应该大写，例如 MyFirstJavaClass 。
- **方法名**：所有的方法名都应该以小写字母开头。如果方法名含有若干单词，则后面的每个单词首字母大写。
- **源文件名**：源文件名必须和类名相同。当保存文件的时候，你应该使用类名作为文件名保存（切记Java是大小写敏感的），文件名的后缀为.java。（如果文件名和类名不相同则会导致编译错误）。
- **主方法入口**：所有的Java 程序由public static void main(String args[])方法开始执行。

### Java修饰符
主要有两类修饰符：
- 可访问修饰符 : default, public , protected, private
- 不可访问修饰符 : final, abstract, strictfp