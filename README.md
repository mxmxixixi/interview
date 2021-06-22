### JavaScript

#### 第一题 什么是防抖和节流，他们的应用场景有哪些，如何实现

> 防抖与节流都是前端优化的方案，使用后可以节省前端消耗的资源，针对不用的场景采用合适的方案

- 防抖[debounce]**是高频事件触发后，n秒内只执行一次**，连续触发就会发生抖动，**为了防止抖动会在连续触发中取消上一次事件**，直达间隔时间达到n秒开始执行事件，总结来说就是**防抖取连续触发的最后一次动作**

- 实现逻辑：闭包实现，自定义Hook实现

  ```javascript
  //闭包实现逻辑：闭包中存储的变量不会被回收
  const debounce = (fn,time)=>{
  	let timeOut = null;
    return (paramter)=>{
      if(timeOut){
        clearTimeout(timeOut)
      }
      timeOut = setTimeout(()=>{
        fn(paramter)
      },time)
    }
  }
  const inputChange = debounce(()=>{console.log('debounce')},500)
  inputChange()
  //自定义hook实现逻辑：useEffect会在下次执行时先执行return函数
  //使用值触发防抖还是函数触发防抖主要看在防抖过程中，中间值是否需要在页面上显示，例如搜索框的需要时时展示在页面上，所以使用值改变，触发防抖
  //值快速更改触发防抖
  const useDebounce = (value,time)=>{
    const [debounceValue,setDebounceValue]=useState(value)
  	useEffect(()=>{
      const timeOut = setTimeoout(()=>{
        setDebounceValue(value)
      },time)
      return ()=>{
        clearTimeout(timeOut)
      }
    },[value,time])
    return debounceValue
  }
  //函数快速执行，触发防抖
  const useDebounceFn = (fn,time,dep=[])=>{
    const { current } = useRef({fn,timeOut:null})//使用useRef进行变量timeOut的记录
  	useEffect(()=>{
      current.fn = fn
    },[fn])
    return useCallback((param)=>{
      if(current.timeOut){
        clearTimeout(current.timeOut)
      }
      current.timeOut = setTimeout(()=>{
        current.fn(param)
      },time)
    },dep)
  }
  ```
  
- 使用场景：所有在高频触发中取最后一次的结果都可以使用防抖【用户搜索、mousedown、mousemove、 keyup、keydown、调整窗口大小事件resize】

<hr />

- 节流[throttle]是在高频事件触发中，n秒内只执行一次，节流会稀释函数的执行频率，总结来说**节流是在高频触发中取第一次进行执行，隔n秒再次执行**

- 实现逻辑：通过闭包

  ```javascript
  //闭包实现
  const throttle = (fn,time)=>{
    let timer = null
    return (param)=>{
      if(timer){
        return
      }
      //fn(param) fn的触发放在定时函数里面也可以，外面直接执行也可以
      timer = setTimeout(()=>{
      	fn(param)  
        time = null
      },time)
    }
  }
  //hook实现，借助useRef缓存数据
  const useThrottle = (value,time)=>{
    const [throttleValue,setThrottleValue] = useState(value)
    const { current } = useRef({timer:null})
    useEffect(()=>{
      if(current.timer){
        return
      }
      current.timer = setTimeout(()=>{
        setThrottleValue(value)
        current.time = null
      },time)
    },[value,time])
    return throttleValue
  }
  const useThrottleFn = (fn,time,dep=[])=>{
    const { current } = useRef({fn,timer:null})
    useEffect(()=>{
      current.fn = fn
    },[fn])
    return useCallback((param)=>{
      if(current.timer){
        return
      }
      current.timer = setTimeout(()=>{
        fn(param)
        current.time = null
      },time)
    },dep)
  }
  ```

- 使用场景：所有高频事件触发中执行第一次【图片下拉加载、滚动页面】

#### 第二题 Promise相关

#### 第三题 ['1', '2', '3'].map(parseInt)

- 返回[1,NaN,NaN]
- **[[1,2,3].map((num,index)=>parseInt(num,index))**
- parseInt() 函数可解析一个字符串，并返回一个整数。parseInt(string, radix):string必需,要被解析的字符串;radix:可选。表示要解析的数字的基数。该值介于 2 ~ 36 之间。如果省略该参数或其值为0，则数字将以10进制数来解析。如果它以 “0x” 或 “0X” 开头，将以16进制数解析。如果该参数小于 2 或者大于 36，则 parseInt()将返回 NaN。字符串string中的数字不能大于radix才能正确返回数字结果值
- const arr = [10,0,10,20,30,40,50,60,70].map((num,index)=>parseInt(num,index))//[10, NaN, 2, 6, 12, 20, 30, 42, 56]

#### 第四道 介绍下 Set、Map、WeakSet 和 WeakMap 的区别

- Set是类数组无重复元素的结构，所有可以迭代的元素都可以转成Set结构，同时Set结构也能转为数组结构，key与value值相同
- Map是一个key可以为任何值，value可以为任何值的对象，可以转换为数组，以及对象，相对比对象结构会更加灵活
- WeakSet 和 WeakMap都是对对象的弱引用，WeakSet他的成员值只能是对象，WeakMap的key值只能是对象，都不可以被遍历

#### 第五道  下面代码中 a 在什么情况下会打印 1？

- 主要考点，在a与1 2 3比较的过程中会出现隐式类型的转换，在比较过程中改变a的值，来达到满足所有的条件

```javascript
var a = ?;
if(a == 1 && a == 2 && a == 3){
 	console.log(1);
}

//第一种：对象的隐式类型转换，转换类型会进行调用toString方法、valueOf方法
var a = {
  i:1,
  toString:()=>{
    return a.i++
  }
}
var a = {
  i:1,
  valueOf:()=>{
    return a.i++
  }
}
//第二种：数组的隐式类型转换，转换是会进行调toString、join方法
//把shift方法赋值给toString方法，每次调用toString都会走shift方法
let a = [1,2,3];
a.toString = a.shift;


let a = [1,2,3];
a.join = a.shift;
//第三种：Symbol.toPrimitive
let a = {[Symbol.toPrimitive]:((i)=>()=>i++)(0)}
```

#### 第六道 函数作用域，var变量

```javascript
var a = 10;
(function () {
    console.log(a)
    a = 5
 		console.log(a)
    console.log(window.a)
    var a = 20;
    console.log(a)
})()
//undefined 10 20
// 1. 立即执行函数内的 var a = 20; 会先进行变量提升。 var a = undefined;会提升到函数作用域顶部
// 2. 这时候console.log(a) 的a为undefined
//  a = 5,此时函数作用域中的a被赋值5，
// 3. console.log(window.a)，这时候取的是window的a变量，全局作用域下的var定义的变量会挂载到window上
// 4. 运行到var a = 20; 这时候会在当前作用域查找a变量，找到了以后赋值为20
// 5. 这时候输出console.log(a) 这时候a的值为20



var a = 10;
(function () {
    console.log(a)
    a = 5
    console.log(window.a)
    console.log(a)
})()
//10 5 5
//演变后的函数作用域中没有定义a变量，直接a=5修改全局的变量a
```

#### 第七道 sort() 排序

- sort()默认情况下按照升序排列数组项，为了实现排序，sort()方法会调用每个数组项的toString(),然后得到比较的字符串；改变原数组，不是纯函数

- sort()在进行比较字符串时，数字无法正常排序，因此sort()方法可以接受一个比较函数作为参数，以便我们指定哪个值在哪个值的前面；比较函数接收两个参数(默认升序)，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回0，如果第一个参数应该位于第二个之后则返回一个正数。

```javascript
//使用 sort() 对数组 [3, 15, 8, 29, 102, 22] 进行排序，输出结果
返回结果：[102,15,22,29,3,8]

```

#### 第八道 未解答

```javascript
var obj = {
    '2': 3,
    '3': 4,
    'length': 2,
    'splice': Array.prototype.splice,
    'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
```

#### 第九道 call 和 apply 的区别是什么，哪个性能更好一些

-  call 和 apply 都是用来改变this的指向，函数接受的第一个参数都是代表this
- call接收this参数和多个其他参数
- apply接受this参数和一个数组
- 使用es6的解构赋值call可以替代apply
- call 的性能要比 apply 好一些（尤其是传递给函数的参数超过三个的时候）
  - apply 的源码执行比较复杂
  - 由于 apply 中定义的参数格式（数组），使得被调用之后需要做更多的事

#### 第十道 实现 (5).add(3).minus(2) 功能

- 主要考在Number.prototype进行添加函数

```javascript
 function addmin(){
   function add(n){
     return this+n
   }
   function minus(n){
     return this-n
   }
   Number.prototype.add = add
   Number.prototype.minus = minus
 }
addmin()
console.log((5).add(3).minus(2))
```

#### 第十一道 **var** a = {n: 1}; **var** b = a; a.x = a = {n: 2}; console.log(a.x) 	 console.log(b.x)

- 引用类型的赋值时地址引用
- 赋值运算符与.运算符的优先级：.运算符的优先级高于赋值运算符

```javascript
var a = {n: 1};
var b = a;//此时a、b都指向堆中的地址
a.x = a = {n: 2};//先进行赋值a.x的值,之后变量a背更改新的堆地址指向，a = {n: 2},b= {n: 1,x:{n:2}}
console.log(a.x)//undefined
console.log(b.x)//{n:2}
```

#### 第十二道 某公司 1 到 12 月份的销售额存在一个对象里面，如下：{1:222, 2:123, 5:888}，请把数据处理为如下结构：[222, 123, null, null, 888, null, null, null, null, null, null, null]。

```javascript
const data = []
const obj = {1:222, 2:123, 5:888}
for(let i=1;i<=12;i++){
  data.push(obj[i] || null )
}

let obj = {1:222, 2:123, 5:888};
const result = Array.from({ length: 12 }).map((_, index) => obj[index + 1] || null);
```

#### 第十三道 箭头函数与普通函数（function）的区别是什么？构造函数（function）可以使用 new 生成实例，那么箭头函数可以吗？为什么？

- 箭头函数与普通函数写法不同，箭头函数是普通函数的简写
- 方法中的this指向不同，箭头函数的this永远取父级作用域的值，普通函数的this取决于它的执行时机
- 箭头函数不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替
- 箭头函数不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。
- 箭头函数不可以使用new生成实例，生成实例的过程，函数的this会指向实例，箭头函数的this已经固定
- 箭头函数没有 prototype 属性 ，而 new 命令在执行时需要将构造函数的 prototype 赋值给新的对象的_proto__
