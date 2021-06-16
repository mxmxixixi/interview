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
