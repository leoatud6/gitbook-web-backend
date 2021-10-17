# onlinestore2021\_基础2\_front-end

* VSCode
* ES6  (same to JDK)
* Node js (npm = maven)
* Vue (spring mvc)
* Babel
* Webpack (packing)

![comparion](<../.gitbook/assets/image (58).png>)

## Vue

MVVM

![](<../.gitbook/assets/image (59).png>)

Dom --> rendering to View

**ViewModel**： core

```bash
#init npm/maven sturture project
npm init -y
npm install vue

#quote and import vue js
<script src="./node_modules/vue/dist/vue.js"></script>

#create Vue
#binding with model: id="xxx"
#{{xx}} : use vue
#v-on / v-model / self-define
#el, data, methods
<div id="chenliu">
    <input type="text" v-model="num">
    <button v-on:click="num++"> thumb</button>
    <button v-on:click="cancel">cancel</button>
    <h1>{{name}}, is beatiful, have {{num}} babeis</h1>
</div>


<script>
let vm = new Vue({
    el:"#chenliu",
    data:{
        name:"caicai",
        num: 2
    },
    methods:{
        cancel(){
            this.num--;
        }
    }
});
</script>

#install vue2 snippet plugin on vscode
#install vue extension on chrome


<spam v-html="msg"></spam>
<spam v-text="msg"></spam>


===v-bind: single direction binding===
#{{}}只可以用在标签体里面，不可以用在标签上
<a v-bind:href="link"> go to this link</a>

<span v-bind:class="{active:isActive, 'text-danger':hasError}"
    v-bind:style="{color:color1,fontSize: size}">hello world!!!</span>


===v-model: dual binding===
#define lang in data in vue, get the checked boxes
<input type="checkbox" v-model="lang" value="java"> java <br>
<input type="checkbox" v-model="lang"  value="php"> php <br>
<input type="checkbox" v-model="lang"  value="c"> c <br>

#display the choices
your choices:<br/>
{{lang.join(",")}}

===v-on===
<input type="text" v-model="num" v-on:keyup.up="num+=2" @keyup.down="num-=2"
@click.ctrl="num=10"><br/>

===v-for===


===v-if / v-else-if===
<button v-on:click="random=Math.random()">click me!!!!!!!</button>
    <span>random number is {{random}}</span>
    <h1 v-if="random>=0.75">&gt; = 0.75</h1>
    <h1 v-else-if="random>=0.6">&gt; = 0.6</h1>
    <h1 v-else-if="random>=0.5">&gt; = 0.5</h1>
    <h1 v-else="random>=0.25">&gt; = 0.25</h1>

=========================in the define area=======================
===computed===
computed:{
    totalPrice(){
        return this.a*asize + b*bsize;
    }
}

===watch===
watch: {
    afunction(newVal, oldVal){
        alert("newVal" + newVal + "==>oldVal" + oldVal)
        if(newVal > 3){
            this.msg = "xxx";
            this.a = 3;
        }else{
            return "xxx";
        }
    }
},

===filter===
#usage
{{user.gender | getGender}}

filter:{
    getGender(val){
        if(val==1){
            return "male";
        }else{
            return "female";
        }
    }
}
```

### 双向binding

### 组件化

将多次使用的部分变成template, 这个vue的名字即为**tag name**

组件不是跟page binding，因此不用el, 用**template**

**data** must be a function， instead of object

![](<../.gitbook/assets/image (60).png>)

### 生命周期

**new Vue** --> many hook functions

* before create
* created
* mount
* mounted
* before update (operation)
* updated (operation)

![](<../.gitbook/assets/image (61).png>)

Three basic elements:

* template
* script
* style

## WebPack 前端脚手架

```bash
npm install webpack -g
npm install -g @vue/cli-init
vue init webpack appname;
npm start
npm run dev
```

Hierarchy

* build folder: webpack
* config folder: configuration, like server port
* node_modules: npm modules
* **src**: source ------> core (index.js, main.js, App.vue, xxx.vue, Router)
* static: 
* **index.html**
* **package.json:** 类似pom.xml

```javascript
//template====================
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>


//script====================
<script>
export default {
  name: 'App'
}
</script>

//style====================
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

#### element UI --> 类似angular?组件？！

