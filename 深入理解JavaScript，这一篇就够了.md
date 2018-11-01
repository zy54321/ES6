[深入理解JavaScript，这一篇就够了](https://www.cnblogs.com/feng-gamer/p/5907926.html)
[toc]

### 对象和原型链
JavaScript 是一门基于对象的编程语言，在 JavaScript 中一切都是对象，包括函数，也是被当成第一等的对象对待，这正是 JavaScript 极其富有表现力的原因。在 JavaScript 中，创建一个对象可以这么写：
```
var someThing = new Object();
```
　　这和在其它面向对象的语言中使用某个类的构造函数创建一个对象是一模一样的。但是在 JavaScript 中，这不是最推荐的写法，使用对象字面量来定义一个对象更简洁，如下：
```
var anotherThing = {};
```
　　这两个语句其本质是一样的，都是生成一个空对象。对象字面量也可以用来写数组以及更加复杂的对象，这样：
```
var weekDays = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
```
　　这样：
```
var person = {
    name : "youxia",
    age : 30,
    gender : "male",
    sayHello : function(){ return "Hello, my name is " + this.name; }
}
```
　　甚至这样数组和对象互相嵌套：
```
var workers = [{name : "somebody", speciality : "Java"}, {name : "another", speciality : ["HTML", "CSS", "JavaScript"]}];
```
　　需要注意的是，对象字面量中的分隔符都是逗号而不是分号，而且即使 JavaScript 对象字面量的写法和 JSON 的格式相似度很高，但是它们还是有本质的区别的。

　　在我们捣鼓 JavaScript 的过程中，工具是非常重要的。我这里介绍的第一个工具就是 Chromium 浏览器中自带的 JavaScript 控制台。在 Ubuntu 中安装 Chromium 浏览器只需要一个命令就可以搞定，如下：
```
sudo apt-get install chromium
```
　　启动 Chromium 浏览器后，只需要按 F12 就可以调出 JavaScript 控制台。当然，在菜单中找出来也可以。下面，让我把上面的示例代码输入到 JavaScript 控制台中，一是可以看看我们写的代码是否有语法错误，二是可以看看 JavaScript 对象的真面目。如下图：
　　
![image](https://images2015.cnblogs.com/blog/16576/201609/16576-20160905213836238-96163993.png)
