# ES6 类(Class)的继承(extends)和自定义存(setter)取值(getter)详解

[ES6 类(Class)的继承(extends)和自定义存(setter)取值(getter)详解](http://blog.csdn.net/pcaxb/article/details/53784309)

原文地址：<http://blog.csdn.net/pcaxb/article/details/53784309>

## 1.类的super方法

子类必须在constructor方法中调用super方法，否则新建实例时会报错。如果子类在constructor方法中使用了this初始化实例属性,调用super方法必须放到this初始化实例属性的前面。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。

```javascript
//ExtendStu.js
//正确   构造
constructor(name, age, height) {
    super(name,age);
    this.height = height;
}
 
//错误  构造
constructor(name, age, height) {
    this.height = height;
    super(name,age);
    //SyntaxError: 'this' is not allowed before super()
}

```

ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。如果子类没有定义constructor方法，这个方法会被默认添加。
​	(1)super作为函数时，指向父类的构造函数。super()只能用在子类的构造函数之中，用在其他地方就会报错。
注意:super虽然代表了父类ExtendStuParent的构造函数，但是返回的是子类ExtendStu的实例，即super内部的this指的是ExtendStu，因此super()在这里相当于ExtendStuParent.prototype.constructor.call(this)。
​	(2)super作为对象时，指向父类的原型对象。

```javascript
toString(){
    return "name:"+ super.getName() + " age:"+this.age + " height:"+this.height;
}

```

上面代码中，子类ExtendStu当中的super.getName()，就是将super当作一个对象使用。这时，super指向ExtendStuParent.prototype，所以super.getName()就相当于ExtendStuParent.prototype.getName()。
由于super指向父类的原型对象，所以定义在父类实例上的属性，是无法通过super调用的。

```javascript
toString(){
    console.log(super.age);//undefined
    return "name:"+ super.getName() + " age:"+this.age + " height:"+this.height;
}
```

age是父类的实例属性,子类通过是super不能调用

```javascript
//ExtendStuParent.js
ExtendStuParent.prototype.width = '30cm';
//ExtendStu.js
toString(){
    console.log(super.age);//undefined
    console.log(super.width);//30cm
    return "name:"+ super.getName() + " age:"+this.age + " height:"+this.height;
}
```

width定义在父类的原型对象上，所以width可以通过super取到。

ES6 规定，通过super调用父类的方法时，super会绑定子类的this。由于绑定子类的this，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性。

```javascript
// 构造
constructor(name, age, height) {
    super(name,age);
    this.height = height;
    //父类和子类都有实例属性name,但是在toString方法调用getName的时候,
    //返回的是子类的的数据
    this.name = 'LiSi 子类';

//
this.color = 'black';
super.color = 'white';
console.log(super.color);//undefined
console.log(this.color);//black

}
```

最后，由于对象总是继承其他对象的，所以可以在任意一个对象中，使用super关键字。

```javascript
import ExtendStuParent from './ExtendStuParent';//父类
import ExtendStu from './ExtendStu';//子类
let extendStu = new ExtendStu('LiSi',12,'170cm');
console.log(extendStu.toString());
console.log(extendStu instanceof ExtendStuParent);//true
console.log(extendStu instanceof ExtendStu);//true
```

说明;ExtendStu继承ExtendStuParent之后,ExtendStu创建的对象同时是ExtendStu和ExtendStuParent两个类的实例。

## 2.类的prototype属性和__proto__属性
Class作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。
(1)子类的__proto__属性，表示构造函数的继承，总是指向父类。
(2)子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。

```javascript
console.log(ExtendStu.__proto__ === ExtendStuParent);//true
console.log(ExtendStu.prototype.__proto__ === ExtendStuParent.prototype);//true
//继承的原理
// ExtendStu的实例继承ExtendStuParent的实例
Object.setPrototypeOf(ExtendStu.prototype, ExtendStuParent.prototype);
// ExtendStu的实例继承ExtendStuParent的静态属性
Object.setPrototypeOf(ExtendStu, ExtendStuParent);
```



作为一个对象，子类（ExtendStu）的原型（__proto__属性）是父类（ExtendStuParent）；作为一个构造函数，子类（ExtendStu）的原型（prototype属性）是父类的实例。

```javascript
console.log(ExtendStuParent.__proto__ === Function.prototype);//true
console.log(ExtendStuParent.prototype.__proto__ === Object.prototype);//true
```



ExtendStuParent作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承Funciton.prototype。但是，ExtendStuParent调用后返回一个空对象（即Object实例），所以ExtendStuParent.prototype.__proto__指向构造函数（Object）的prototype属性。

```javascript
console.log(Object.getPrototypeOf(ExtendStu) === ExtendStuParent  );//true
```



Object.getPrototypeOf方法可以用来从子类上获取父类。因此，可以使用这个方法判断，一个类是否继承了另一个类。

## 3.实例的__proto__属性
子类实例的__proto__属性的__proto__属性，指向父类实例的__proto__属性。也就是说，子类的原型的原型，是父类的原型。

```javascript
class A{}
class B extends A{}
let a = new A();
let b = new B();
console.log(b.__proto__ === a.__proto__);//false
console.log(b.__proto__.__proto__ === a.__proto__);//true
```



因此，通过子类实例的__proto__.__proto__属性，可以修改父类实例的行为。

```javascript
b.__proto__.__proto__.getName = ()=>{
    console.log('给父类添加一个函数');
};
b.getName();//给父类添加一个函数
a.getName();//给父类添加一个函数
```



说明:使用子类的实例给父类添加getName方法,由于B继承了A,所以B和A中都添加了getName方法。

## 4.原生构造函数的继承
原生构造函数是指语言内置的构造函数，通常用来生成数据结构。ECMAScript的原生构造函数大致有下面这些。
**Boolean()**
**Number()**
**String()**
**Array()**
**Date()**
**Function()**
**RegExp()**
**Error()**
**Object()**
以前，这些原生构造函数是无法继承的，比如，不能自己定义一个Array的子类。

```javascript
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});
```

上面代码定义了一个继承Array的MyArray类。但是，这个类的行为与Array完全不一致。

```javascript
var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```



之所以会发生这种情况，是因为子类无法获得原生构造函数的内部属性，通过Array.apply()或者分配给原型对象都不行。原生构造函数会忽略apply方法传入的this，也就是说，原生构造函数的this无法绑定，导致拿不到内部属性。ES5是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的内部属性无法获取，导致无法继承原生的构造函数。比如，Array构造函数有一个内部属性[[DefineOwnProperty]]，用来定义新属性时，更新length属性，这个内部属性无法在子类获取，导致子类的length属性行为不正常。
ES6允许继承原生构造函数定义子类，因为ES6是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。下面是一个继承Array的例子。

```javascript
//ExtendsArray.js
class ExtendsArray extends Array{}

let extendsArray = new ExtendsArray();
console.log("=====ExtendsArray=====");
extendsArray[0] = '数据1';
console.log(extendsArray.length);
```



上面代码定义了一个ExtendsArray类，继承了Array构造函数，因此就可以从ExtendsArray生成数组的实例。这意味着，ES6可以自定义原生数据结构（比如Array、String等）的子类，这是ES5无法做到的。上面这个例子也说明，extends关键字不仅可以用来继承类，还可以用来继承原生的构造函数。因此可以在原生数据结构的基础上，定义自己的数据结构。

## 5.Class的取值函数（getter）和存值函数（setter）
取值函数（getter）和存值函数（setter）可以自定义赋值和取值行为。当一个属性只有getter没有setter的时候，我们是无法进行赋值操作的,第一次初始化也不行。如果把变量定义为私有的(定义在类的外面),就可以只使用getter不使用setter。

```javascript
//GetSet.js
let data = {};

class GetSet{

// 构造
constructor() {
    this.width = 10;
    this.height = 20;
}
 
get width(){
    console.log('获取宽度');
    return this._width;
}
 
set width(width){
    console.log('设置宽度');
    this._width = width;
}
//当一个属性只有getter没有setter的时候，我们是无法进行赋值操作的,第一次初始化也不行。
//bundle.js:8631 Uncaught TypeError: Cannot set property width of #<GetSet> which has only a getter

get data(){
    return data;
}
//如果把变量定义为私有的,就可以只使用getter不使用setter。

}

let getSet = new GetSet();
console.log("=====GetSet=====");
console.log(getSet.width);
getSet.width = 100;
console.log(getSet.width);
console.log(getSet._width);
console.log(getSet.data);

//存值函数和取值函数是设置在属性的descriptor对象上的。
var descriptor = Object.getOwnPropertyDescriptor(GetSet.prototype,"width");
console.log("get" in descriptor);//true
console.log("set" in descriptor);//true
```



如果把上面的get和set改成以下形式,不加下划线报Uncaught RangeError: Maximum call stack size exceeded错误。这是因为,在构造函数中执行this.name=name的时候，就会去调用set name，在set name方法中，我们又执行this.name = name，进行无限递归，最后导致栈溢出(RangeError)。

```javascript
get width(){
    console.log('获取宽度');
    return this.width;
}

set width(width){
    console.log('设置宽度');
    this.width = width;
}
```



说明:以上width的getter和setter只是给width自定义存取值行为,开发者还是可以通过_width绕过getter和setter获取width的值。

## 6.new.target属性
new是从构造函数生成实例的命令。ES6为new命令引入了一个new.target属性，（在构造函数中）返回new命令作用于的那个构造函数。如果构造函数不是通过new命令调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

```javascript
class Targettu{
    // 构造
    constructor() {
        if(new.target !== undefined){
            this.height = 10;
        }else{
            throw new Error('必须通过new命令创建对象');
        }
    }
}
//或者(判断是不是通过new命令创建的对象下面这种方式也是可以的)
class Targettu{
    // 构造
    constructor() {
        if(new.target === Targettu){
            this.height = 10;
        }else{
            throw new Error('必须通过new命令创建对象');
        }
    }
}

let targettu = new Targettu();
//Uncaught Error: 必须通过new命令创建对象
//let targettuOne = Targettu.call(targettu);
```



需要注意的是，子类继承父类时，new.target会返回子类。通过这个原理,可以写出不能独立使用、必须继承后才能使用的类。

```javascript
// 构造
constructor(name,age) {
    //用于写不能实例化的父类
    if (new.target === ExtendStuParent) {
        throw new Error('ExtendStuParent不能实例化,只能继承使用。');
    }
    this.name = name;
    this.age = age;
}
```



## 7.Mixin模式的实现
Mixin模式指的是，将多个类的接口“混入”（mix in）另一个类。它在ES6的实现如下。

```javascript
//MixinStu.js
export default function mix(...mixins) {
    class Mix {}
 
    for (let mixin of mixins) {
        copyProperties(Mix, mixin);
        copyProperties(Mix.prototype, mixin.prototype);
    }
 
    return Mix;
}
 
function copyProperties(target, source) {
    for (let key of Reflect.ownKeys(source)) {
        if ( key !== "constructor"
            && key !== "prototype"
            && key !== "name"
        ) {
            let desc = Object.getOwnPropertyDescriptor(source, key);
            Object.defineProperty(target, key, desc);
        }
    }
}

```



上面代码的mix函数，可以将多个对象合成为一个类。使用的时候，只要继承这个类即可。

```javascript
class MixinAll extends MixinStu(B,Serializable){
 
    // 构造
    constructor(x,y,z) {
        super(x,y);
        this.z = z;
    }
 
}

```



更多的资料:点击打开链接   点击打开链接

Class的继承(extends)和自定义存值(setter)取值(getter)的整个案例:

```javascript
//ExtendStuParent.js
const ExtendStuParent = class ExtendStuParent{
 
    name;//定义name
    age;//定义age
 
    // 构造
    constructor(name,age) {
        //用于写不能实例化的父类
        if (new.target === ExtendStuParent) {
            throw new Error('ExtendStuParent不能实例化,只能继承使用。');
        }
        this.name = name;
        this.age = age;
    }
 
    getName(){
        return this.name;
    }
 
    getAge(){
        return this.age;
    }
};
 
ExtendStuParent.prototype.width = '30cm';
 
export default ExtendStuParent;
 
//ExtendStu.js
import ExtendStuParent from './ExtendStuParent';
 
export default class ExtendStu extends ExtendStuParent{
 
    // 构造
    constructor(name, age, height) {
        super(name,age);
        this.height = height;
        //父类和子类都有实例属性name,但是在toString方法调用getName的时候,
        //返回的是子类的的数据
        this.name = 'LiSi 子类';
 
        //
        this.color = 'black';
        super.color = 'white';
        console.log(super.color);//undefined
        console.log(this.color);//black
    }
 
    toString(){
        console.log(super.age);//undefined
        console.log(super.width);//30cm
        return "name:"+ super.getName() + " age:"+this.age + " height:"+this.height;
    }
 
}

```



参考资料:点击打开链接


ES6 类(Class)的继承(extends)和自定义存(setter)取值(getter)详解

博客地址：http://blog.csdn.net/pcaxb/article/details/53784309

下载地址：http://download.csdn.net/detail/pcaxb/9717587