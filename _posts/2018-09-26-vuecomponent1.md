---
layout: post
title: 咱编写的常用Vue组件1（数字拨调输入框）
date: 2018-09-26 
tag: Vue.js
---
 # Component说明  
 &nbsp; &nbsp;为Vue练习而编写的一个常用组件。可以实现在有Vue实例的地方使用<input-value></input-value>标签生成一个可以点击“+”“-”按钮调节数字大小的功能。输入框中只能填写数字，如果输入的不是数组会在出现一个警告框提醒，之后在控制台抛出一个错误，提示输入的并不是数字。  
 &nbsp; &nbsp;还支持从父组件设置一个输入框的默认值。直接Copy的话默认值是5。输入框中的数据在任何时刻的任何改动都会与父组件中设置的值（value）保持一致。因为组件简单且样式不固定，不使用CSS加以修饰。  
 &nbsp; &nbsp;以下是我的鄙陋代码：  
 v4.html：
 ```
   <div id = "app1">
        <input-number :value = "value" @input="getNewVal"></input-number>
    </div>
    <template id = "input-number">
        <div>
            <input type = "text" v-model = "currentValue">
            <input type = "button" value = "+" @click="add" >
            <input type = "button" value = "-" @click = "red">
        </div>
    </template>
 ```
 v4.js
 ```
 
function isNumberValue(value){ // 判断参数是否为数字
    return (/(^-?[0-9]+\.{1}\d+$)|(^-?[1-9][0-9]*$)|(^-?0{1}$)/).test(value + ""); 
}

Vue.component("input-number",{
    template:"#input-number",
    props:{
        max:{
            type:Number,
            default:365 // 可在父组件中传值顶替这个子组件内部的默认值
        },
        min:{
            type:Number,
            default: 0 // 可在父组件中传值顶替这个子组件内部的默认值
        },
        value:{
            type:Number,
            default:Infinity
        }   
    },
    data:function(){
        return {
            currentValue:this.value
        }
    },
    methods:{
        add:function(){
           this.currentValue += 1; 
           this.check(this.currentValue);
           this.$emit("input",this.currentValue);
        },
        red:function(){
            this.currentValue -= 1;
            this.check(this.currentValue);
            this.$emit("input",this.currentValue);
        },
        check:function(val){ // 检测输入的数字是否超值并且是否为数字
            var val = Number(val); // 修改输入框中的内容默认是字符串形式需要转换成Number
            if(isNumberValue(this.currentValue)){
                if(val > this.max) { this.currentValue = this.max; console.log("数字过大"); return; }
                else if(val < this.min) { this.currentValue = this.min; console.log("数字过小"); return; }
                else {
                    this.currentValue = val;
                }
            } else {
                alert("输入数字好吗");
            }
            
        },
        coin:function(){ // 用来测试是否可以成功调用函数的函数样品
            console.log(this.currentValue);
        }
        

    },
    watch:{
        currentValue:function(newValue,oldValue){
            console.log(newValue + "||" + oldValue);
            if(newValue == "")
            {
                return;
            } else {
                this.check(newValue);    
            }
            this.$emit("input",this.currentValue);
        }
    },
    mounted:function(){
        this.check(this.value); // 检测父组件的默认值是否超值
    },
    
});


vm = new Vue({
    el:"#app1",
    data:{
        value:5, // 默认值
        max:365, 
        min:0
    },
    methods:{
        getNewVal:function(val){
            this.value = val;
        }
    }
});
// 1.0 定义模板
// 2.0 在父组件环境设置默认值,所有值只能是数字
// 3.0 按加减时获取全新的值（新输入的值进行处理后的值），并把全新的值传给父元素的value
 ```
 
