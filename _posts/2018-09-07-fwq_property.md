---
layout: post
title: Javascript中的控制属性的行为方式/数据属性&访问器属性
date: 2018-09-07
tag: Javascript
---

# Javascript中的控制属性的行为方式/数据属性&访问器属性

&emsp;&emsp;最近发现了一个有趣的东西，让我了解了属性的属性（这么说不知道贴不贴切）。
这些小东西看上去可能并不是很重要，并且大多数情况可能都不会用到这样高级的属性。但是，我认为了解这些东西还是有的，像是在我们枚举某个对象的一些属性的时候，我们可能不希望让所有的属性都出现在我们的眼前，或是我们希望这个属性不会被不小心的删除或者重置。这些
小属性就派上了用场。这些[[属性]]是为了实现Javascript引擎用的，但是却不能被Javascript引擎直接访问，
而是需要通过特定的函数才能访问。为了表示这些属性是内布值，ECMA-262规范把他们放在了两对方括号中。

&emsp;&emsp;ECMAScript中有两种属性：**数据属性** 和 **访问器属性**

### 数据属性
&emsp;&emsp;我认为数据属性就是那些保存着一些值的属性，他们的最大特点是拥有[[Value]],属性
 这个属性中保存着值。  
  &emsp;&emsp;数据属性包含着四个属性:  
  &emsp;&emsp; &emsp;&emsp;[[Configurable]]：false就不会被delete所删除，也不能把他更改为访问器属性。  
  &emsp;&emsp; &emsp;&emsp;[[Enumerable]]:false不能通过for-in循环返回。  
  &emsp;&emsp; &emsp;&emsp;[[Writeable]]:false则不能修改属性的值。  
  &emsp;&emsp; &emsp;&emsp;[[Value]]：这个属性中保存的就是这个属性的值。

## 访问器属性  
 &emsp;&emsp;访问器属性和数据属性的区别在于，他们并不含有特定的数据值，
  因此，他们并不含有[[Value]]属性，他们的“值”只是一对儿get和set函数，在读取访问器属
  性的时候会调用get函数，在写入数据的时候则会调用set函数。这就可以让我们
  很方便的在为函数写入值的时候自动执行一些逻辑，而不用在另创建一个函数来根据属性的值
  来执行其他的逻辑。
  ```
  var book = {};
  Object.defineProperties(book,{
      year:{
          enumerable:true,
          /**
           在ECM5中enumerable属性的默认值是true，但是在ECM6中，默认值是false,
           (真的很烦啊，标准变来变去的，看来数据属性和访问器属性被特殊点名的时候，
           希望enumerable属性为false的时候占大多数啊)
          **/
          configurable:false, //设置year属性不能被delete删除
          writable:true,
          value:2018,
      },
      edition:{
          enumerable:false,  //设置edition属性不能被for in循环所返回
          writable:true,
          value:1
      }
  });
  delete book.year;
  console.log(book.year); //依旧输出2018

  for(var x in book){
      console.log(book[x]); //并没有输出1
  }

```

另一段代码展示访问器属性，set和get
```
var book = {};
Object.defineProperties(book,{
    _year:{
        enumerable:true, //在ECM5中enumerable属性的默认值是true，但是在ECM6中，默认值是false
        configurable:false, //设置year属性不能被delete删除
        writable:true,
        value:2018,
    },
    edition:{
        enumerable:false,  //设置edition属性不能被for in循环所返回
        writable:true,
        value:0
    },
    year:{
        get:function(){
            return this._year;
        },
        set:function(newYear){
            if(newYear >= this._year){
                this.edition = newYear - this._year;
                this._year = newYear;
            }
        }

    }
});

console.log(book.year + "||" + book.edition); //输出2018 || 0
book.year = 2019;
console.log(book.year + "||" + book.edition); //输出2019 || 1

```

值得注意的是，上面的代码，year属性不在是原来的数据属性_year（一般只有对象内部才能访问到的属性前
面会加一个下划线），成为了一个新的访问器属性。在这个访问器属性中，我们更改了新的edition，同时更
改了_year。如果不这样做，我们就需要一个新的单独的函数来更新edition._year。）

##读取属性的特性（这里才想起来，叫做属性的特性应该更合适）
&emsp;&emsp;很简单，只要使用ECMAScript5的Object.getOwnPropertyDescriptor()方法即可。
```
var descriptor = Object.getOwnPropertyDescriptor("book","_year");
console.log(descriptor.value + "||" + descriptor.configurable);
//2018||false
```
