# Gradle(6) 自定义Task


## 简单的定义一个Task

我们可以把下面这段代码写在 build.gradle文件中，然后我们可以通过：

`gradle loadFile`来执行这个自定义的Task

```
task loadFile(type : Copy, dependsOn : 'distribution') {
    from 'src/main/doc'
    into 'build/target/doc'
    doLast {
        def files = file('../antLoadfileResources').listFiles().sort()
        files.each { File file ->
            if (file.isFile()) {
                ant.loadfile(srcFile: file, property: file.name)
                println " *** $file.name ***"
                println "${ant.properties[file.name]}"
            }
        }
    }
}
```
上面定义了一个比较复杂的Task：loadFile

* 类型为Copy，可以理解为该Task的父类为[Copy](https://docs.gradle.org/4.3.1/javadoc/org/gradle/api/tasks/Copy.html)。
* 依赖于distribution这个task执行，即 distribution Task 执行后，执行 loadFile
* 在该Task执行完毕后会执行 doLast Action。

下面我们来看一下为什么可以这样写，很简单，基于前面的知识，在Project中可以找到这个API：
![](/Users/susion/Documents/JianShuArtical/gradle/自定义Task/simpleDefineTask.png)

![](/Users/susion/Documents/JianShuArtical/gradle/自定义Task/optins.png)

## 复杂Task的定义

```
class GreetingTask extends DefaultTask {
    public String words;
    
    @TaskAction
    def greet() {
        println 'hello from GreetingTask'
    }
}

// Create a task using the task type
task hello(type: GreetingTask){
    words = "Hello ........"
}

```

在build.gradle上面定义这段代码，执行`gradle hello`会打印'Hello......'

一般我们在定义






