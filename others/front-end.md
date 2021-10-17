# Front-End

![](<../.gitbook/assets/image (25).png>)

ES6： ECMAScript6.0   ==> standard 

```javascript
//let: only in its scope
let l;
//const : cannot change
const c;
//var: can be widly use
var v;
console.log(a,b,c);

const{name, age, language};

let str = "hello.vue";
str.startsWith("he");  //--> true/false

function fun(){
    return "haha";
}

//multi input var
function fun1(...values){
    console.log(values.length)
}

// get all string in str
${str} 

//lambda
var sum = function(a,b){
    c = a+b;
    return a + c;
}

var sum2 = (a,b)=>a+b;

var sum3 = (a,b)=>{
    c = a+b;
    return a+c;
};

//lambda + arrow
var hello2 = ({name}) => console.log("hello" + name);
hello2(person);


//map
let arr = ['1','20','-9','0'];

arr =  arr.map((item)=>{
    return item * 2 + 3;
});

arr = arr.map(item => item * 2+3);







```
