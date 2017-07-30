---
uuid: c9c38910-74e4-11e7-82af-11093d25fc7f
author: GodEngine
email: chenggong@blued.com
avatar: https://avatars3.githubusercontent.com/u/16115512?v=4
title: redux中间件
date: 2017-07-30 13:05:58
tags:
---
- [何为中间件](#middleware中间件)
- [redux中间件的意义](#redux中间件的意义)
- [redux的中间件是如何实现的](#redux中间件的实现)
    
## middleware中间件:
一个模拟中间件的小栗子
```javascript
// 两个中间件
function middleware1() {
    console.log('before middleware1')
    console.log('after middleware1')

}

function middleware2() {
    console.log('after middleware2')
    console.log('after middleware2')
}

// 我们的逻辑
function fun(){
    console.log('our fun')
}

// 最终增强后的fun变为如下的wraperFun
function wraperFun() {
    console.log('before middleware1')
    console.log('before middleware2')
    
    console.log('our fun')
    
    console.log('after middleware2')
    console.log('after middleware1')
}
```
以上过程中，wraperFun作为fun 的增强函数，将fun的执行过程包裹在两个中间件的逻辑之间。而我们需要的，就是有这样一个逻辑(即函数)applyMiddleware，输入fun和各个middleware，输出一个增强后的wraperFun，这个applyMiddleware能在执行fun的前后实现我们预先给定的middleware逻辑。

## redux中间件的意义
在发送dispatch(action)修改store状态值的前后，我们需要一些诸如打日志或者实现请求前设置pendding状态和请求后设置success状态等等公共逻辑，这些逻辑即可放在中间件实现。

## redux中间件的实现
redux API提供了一个名为applyMiddleware的方法，该方法就是以上我们提到的applyMiddleware，该方法内部将store的dispatch方法包裹在给定的middleware之间，并将增强后的dispatch挂到原redux的store上，使得我们在调用dispatch(action)时执行中间件的逻辑。看下applyMiddleware的源码：
```javascript
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer)
    var dispatch = store.dispatch
    var chain = []

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)    // 增强dispatch的过程

    return {
      ...store,
      dispatch      // 增强后的dispatch方法
    }
  }
}
```
而这个compose方法内部用Array.prototype.reduceRight方法从右至左将将每一个右侧的中间件执行后的结果（一个dispatch函数）作为左侧中间件内部使用的dispatch函数，这样每个中间件中用到的dispatch都是由右侧的中间件提供的，最右侧的中间件中用到的dispatch由store提供，这个store的dispatch函数被从右向左层层传递到左侧第一个函数中，被这些中间件们包裹起来。接下来看compose源码方法是如何实现的：
```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  const last = funcs[funcs.length - 1]
  const rest = funcs.slice(0, -1)
  return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
}

// 为看的更清晰，以下demo1为模拟上面compose内部运行代码，并假设有两个中间件middlewareReturn1和last，可试运行查看结果

function middlewareReturn1 (next) { // 该middlewareReturn1对应rest中的中间件
    return function (action) {
        console.log('中间件1的逻辑前')
        next(action)
        console.log('中间件1的逻辑后')
    }
}

function last (next) {      // 该last对应compose中的last中间件
    return function (action) {
        console.log('中间件2的逻辑前 ')
        next(action)
        console.log('中间件2的逻辑后 ')
    }
}

var dispath = function(action){   // 该dispatch对应compose中穿给compose的store.dispatch
    console.log('store.dispath: ' + action)
}

var rest=[middlewareReturn1] // 该数组rest对应compose中的rest数组

var finaldispath = rest.reduceRight(function(composed, f){
                    return f(composed)
                }, last(dispath)
)
finaldispath('test')

/**结果为
中间件1的逻辑前
    中间件2的逻辑前 
        store.dispath: test
    中间件2的逻辑后 
中间件1的逻辑后
**/
```
以上可以看到compose函数是如何将中间件串起来的，下面我们来看中间件应该是什么样子的，最典型及最常用的redux中间件redux-chunk的代码，只有区区几行：
```javascript
function createThunkMiddleware (extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument)
    }

    return next(action)
  }
}

const thunk = createThunkMiddleware()
thunk.withExtraArgument = createThunkMiddleware

export default thunk

// 去掉内部的逻辑，中间件应该是这样的
function createThunkMiddleware (extraArgument) {
  return (store) => next => action => {
    next(action)
  }
}

const thunk = createThunkMiddleware()
export default thunk

/**层层拆分下这个middleware**/

//  以下function为传递给applyMiddleware函数的参数
(store) => next => action => {
    next(action)
 } 
  
 // 以下function为applyMiddleware函数中map上面的参数返回的结果，就是数组chain里的元素，
 (next) => action => {
    next(action)
 } 
 
  // 以下function为通过reduceRight函数遍历chain时依次传给下一个chain内元素的参数，这个next遍历，初始值是store.dispatch,每一个chain内元素内部持有的next，都是这个初始值store.dispatch
 action => { 
    next(action)
 }
```
核心其实就是这个reduceRight方法的实现，可以运行下demo1体验下。后期会补上redux-chunk在发起异步action时是如何起作用的。