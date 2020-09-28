---
title: Vue+vue-router+vuex原理分析
date: 2020-05-29 23:11:10
tags:
    - Vue
---

Vue源码中有很多零碎知识点值得学习，本篇博文记录学习Vue中常见API的实现原理。
<!--more-->

> new Vue初始化流程

- 介绍
Vue初始化是
```
new Vue({
    el: '#app',
    data: {
        name: '张三'
    }
})
```
传入的参数是一个object,包含了绑定的根节点`#app`和初始化数据`data`。
查看Vue源码，Vue构造函数执行初始化方法`_init(options)`，`_init`依次执行了以下方法
```
    vm.$options = mergeOptions() // 扩展vue $options方法，该方法可以merge对象属性，扩展了vue属性
    initProxy(vm);  //为vue _renderProxy属性设置proxy代理。这层代理会在模板渲染时对一些非法或者没有定义的变量进行筛选判断
    vm._self = vm;
    initLifecycle(vm);  //添加初始化生命周期属性
    initEvents(vm); //绑定组件上的事件, 涉及到$on，$once, $off, $emit
    initRender(vm); //最主要的方法是defineReactive$$1，通过Object.defineProperty + 观察者模式，实现数据拦截
    callHook(vm, 'beforeCreate');   // 调用生命周期钩子 beforeCreate
    initInjections(vm); // 解析inject属性
    initState(vm);  //初始化props,methods, data,computed,watch属性
    initProvide(vm); // 解析provide属性
    callHook(vm, 'created');    //  调用生命周期钩子 created
    
    ...
    
    if (vm.$options.el) {
        vm.$mount(vm.$options.el);  // 生成template -> 生成render函数 -> 调用beforeMount -> 生成VNode -> vm._update生成真实DOM -> mounted
    }
```



> nextTick

- 介绍
nextTick用于下一次DOM更新执行的方法，当赋值变量之后，DOM节点并没有及时更新，所以此时获取节点内容并不是最新的，因此需要使用nextTick在DOM节点变化之后，获取最新的DOM。

- 原理
nextTick是采用异步控制+优雅降级的方式，以v2.6版本为例，降级策略为根据浏览器兼容性依次判断是否支持Promise -> MutationObserver -> setImmediate -> setTimeout。
其中，Promise和MutationObserver属于微任务队列，setImmediate和setTimeout属于宏任务队列，微任务比宏任务优先执行。
nextTick将回调方法收集到callbacks数组中，在异步任务执行时，将callbacks中的函数依次执行。
例如在使用MutationObserver时，源码中创建了一个node节点，MutationObserver监听此节点内容的变化。当需要执行nextTick的回调函数时，手动触发node节点内容的变化，推动MutationObserver触发监听，执行callbacks中的方法。


> 双向绑定

-   介绍
Vue最基本的功能便是双向绑定，与jQuery相比，极大地方便了DOM操作，可以说是革了jQuery的命。使开发者可以专注于数据操作，数据推动DOM变化。

-   原理
Vue初始化绑定data数据是在initState中的initData
```
function initState (vm) {
    vm._watchers = [];
    var opts = vm.$options;
    if (opts.props) { initProps(vm, opts.props); }
    if (opts.methods) { initMethods(vm, opts.methods); }
    if (opts.data) {
      initData(vm);
    } else {
      observe(vm._data = {}, true /* asRootData */);
    }
    if (opts.computed) { initComputed(vm, opts.computed); }
    if (opts.watch && opts.watch !== nativeWatch) {
      initWatch(vm, opts.watch);
    }
  }
```
`initData`中做了:
1.  判断数据类型是否合规；
2.  判断是否有重名的key,比如props中和data中不能有相同的属性；
3. `proxy(vm, "_data", key);`代理data中的对象到this下。实现可以app.name可以调用app._data.name
4. `observe(data, true /* asRootData */);`绑定数据

再看observe:
```
function observe (value, asRootData) {
    ...
    if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
      ob = value.__ob__;
    } else if (
      shouldObserve &&
      !isServerRendering() &&
      (Array.isArray(value) || isPlainObject(value)) &&
      Object.isExtensible(value) &&
      !value._isVue
    ) {
      ob = new Observer(value);
    }
    ...
  }
```
主要是先判断是否有`__ob__`,如果没有，就把`__ob__`设为new Observer实例。

再看Observer:
```
  var Observer = function Observer (value) {
    this.value = value;
    this.dep = new Dep();   
    this.vmCount = 0;
    def(value, '__ob__', this); //将Observer绑定到`__ob__`下
    if (Array.isArray(value)) {
        ...
      this.observeArray(value);
    } else {
      this.walk(value);
    }
  };
```

`walk`和`observeArray`是针对对象和数组进行数据绑定，数组的话，就循环针对每一个属性进行绑定。
walk:
```
Observer.prototype.walk = function walk (obj) {
    var keys = Object.keys(obj);
    for (var i = 0; i < keys.length; i++) {
      defineReactive$$1(obj, keys[i]);
    }
  };
```
walk就是针对对象中的每一个属性进行循环绑定。
`defineReactive$$1`这里就是使用`Object.defineProperty`的`set`,`get`进行数据绑定，
```
    get: function reactiveGetter () {
        //这里主要是判断用户是否自定义了getter，如有,就是用自定义getter
        var value = getter ? getter.call(obj) : val;
        if (Dep.target) {
          dep.depend();
          if (childOb) {
            childOb.dep.depend();
            if (Array.isArray(value)) {
              dependArray(value);
            }
          }
        }
        return value
      },
      set: function reactiveSetter (newVal) {
      
        var value = getter ? getter.call(obj) : val;
        /* eslint-disable no-self-compare */
        if (newVal === value || (newVal !== newVal && value !== value)) {
          return
        }
        /* eslint-enable no-self-compare */
        if (customSetter) {
          customSetter();
        }
        // #7981: for accessor properties without setter
        if (getter && !setter) { return }
        if (setter) {
          setter.call(obj, newVal);
        } else {
          val = newVal;
        }
        //  新修改的属性，进行数据绑定
        childOb = !shallow && observe(newVal);
        dep.notify();
      }
```
dep用于依赖管理，收集Watcher。它会用subs收集Watcher，当执行setter时，就将subs中的Watcher依次循环，执行每一个Watcher的notify。

总的数据管理流程就是：Observer -> dep -> watcher

> Vue异步更新过程

- 介绍
当js中的变量发生变化时，DOM上绑定变量的节点内容并没有立刻发生变化，而是通过nextTick将tick之间发生的内容变化，一次性更新在DOM上的。

-   原理
以如下代码为例：
```
<div id="app">
  <p @click="handleClick">{{ test }}</p>
</div>
```

```
const app = new Vue({
      el: '#app',
      data:{
          test: '111'
      },
      methods:{
          handleClick(){
              this.test = this.test * 1 + 1;
          }
      }
  })
```
当点击P标签时，会触发`handleClick`,首先要获取`this.test`, Vue是通过`proxy`将`this._data`下的变量代理到this下的。
```
function proxy (target, sourceKey, key) {
    sharedPropertyDefinition.get = function proxyGetter () {
      return this[sourceKey][key]
    };
    sharedPropertyDefinition.set = function proxySetter (val) {
      this[sourceKey][key] = val;
    };
    Object.defineProperty(target, key, sharedPropertyDefinition);
  }
```
proxy的get会调用`Object.defineProperty`的get。
当给test赋新值时，又通过proxy的set调用`Object.defineProperty`的set。
```
set: function reactiveSetter (newVal) {
        
        var value = getter ? getter.call(obj) : val;
        //如果新旧值相同，就不处理
        if (newVal === value || (newVal !== newVal && value !== value)) {
          return
        }
        //如果有自定义的setter, 就去执行自定义的setter
        if (customSetter) {
          customSetter();
        }
        // #7981: for accessor properties without setter
        if (getter && !setter) { return }
        if (setter) {
          setter.call(obj, newVal);
        } else {
          val = newVal;
        }
        //这里如果新的值是对象的话，就把对象中的每一个key重新绑定响应
        childOb = !shallow && observe(newVal);
        // 通知dep中的观察者去更新
        dep.notify();
      }
```

```
Dep.prototype.notify = function notify () {
    // stabilize the subscriber list first
    var subs = this.subs.slice();
    if (!config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort(function (a, b) { return a.id - b.id; });
    }
    for (var i = 0, l = subs.length; i < l; i++) {
      subs[i].update();
    }
  };
```
dep中绑定的Watcher，都在subs中。通知每一个Watcher去执行它们的update。
```
Watcher.prototype.update = function update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true;
    } else if (this.sync) {
      this.run();
    } else {
      queueWatcher(this);
    }
  };
```
update并不是立即去执行更新，而是通过queueWatcher方法把需要更新的事件放进队列
```
  function queueWatcher (watcher) {
    var id = watcher.id;
    if (has[id] == null) {
      has[id] = true;
      if (!flushing) {
        queue.push(watcher);
      } else {
        // if already flushing, splice the watcher based on its id
        // if already past its id, it will be run next immediately.
        var i = queue.length - 1;
        while (i > index && queue[i].id > watcher.id) {
          i--;
        }
        queue.splice(i + 1, 0, watcher);
      }
      // queue the flush
      if (!waiting) {
        waiting = true;

        if (!config.async) {
          flushSchedulerQueue();
          return
        }
        nextTick(flushSchedulerQueue);
      }
    }
  }
```
放进队列之后，就通过nextTick去执行队列里的事件。nextTick前面已经介绍过了，nextTick传了一个回调`flushSchedulerQueue`，Watcher的更新操作是在这个方法中进行
```
  function flushSchedulerQueue () {
    flushing = true;
    var watcher, id;

    // Sort queue before flush.
    // This ensures that:
    // 1. Components are updated from parent to child. (because parent is always
    //    created before the child)
    // 2. A component's user watchers are run before its render watcher (because
    //    user watchers are created before the render watcher)
    // 3. If a component is destroyed during a parent component's watcher run,
    //    its watchers can be skipped.
    /*
    给queue排序，这样做可以保证：
    1.组件更新的顺序是从父组件到子组件的顺序，因为父组件总是比子组件先创建。
    2.一个组件的user watchers比render watcher先运行，因为user watchers往往比render watcher更早创建
    3.如果一个组件在父组件watcher运行期间被销毁，它的watcher执行将被跳过。
  */
    queue.sort(function (a, b) { return a.id - b.id; });

    // do not cache length because more watchers might be pushed
    // as we run existing watchers
    for (index = 0; index < queue.length; index++) {
      watcher = queue[index];
      if (watcher.before) {
        watcher.before();
      }
      id = watcher.id;
      has[id] = null;
      watcher.run();    //执行Watcher
      // in dev build, check and stop circular updates.
      if (has[id] != null) {
        circular[id] = (circular[id] || 0) + 1;
        //如果超出最大更新次数，就给个提示。
        if (circular[id] > MAX_UPDATE_COUNT) {
          warn(
            'You may have an infinite update loop ' + (
              watcher.user
                ? ("in watcher with expression \"" + (watcher.expression) + "\"")
                : "in a component render function."
            ),
            watcher.vm
          );
          break
        }
      }
    }

    // keep copies of post queues before resetting state
    var activatedQueue = activatedChildren.slice();
    var updatedQueue = queue.slice();

    //重置队列状态
    resetSchedulerState();

   //调用生命周期钩子
    callActivatedHooks(activatedQueue);
    callUpdatedHooks(updatedQueue);

    // devtool hook
    /* istanbul ignore if */
    if (devtools && config.devtools) {
      devtools.emit('flush');
    }
  }
```
这样一次tick过程就结束了，总的过程是：setter -> dep.notify -> Watcher.update -> nextTick -> Watcher.run

> patch和diff

-   介绍
Vue在更新DOM时，并不会全部更新DOM，而是把需要更新的节点进行更新，那么他是怎么计算哪些节点需要更新呢，这就要讲到diff算法了。diff算法是一种通过同层的树节点进行比较的高效算法，复杂度只有 O(n)

-   分析
Vue更新操作是在nextTick后执行的`flushCallbacks`，依次去执行监听者Watcher中的run方法，然后执行`vm._update(vm._render(), hydrating)`, 参数`vm._render()`就是需要更新的Vnode。_update中有一句`vm.$el = vm.__patch__(prevVnode, vnode);`，这里就是执行patch方法了。

`patch`中主要是判断新旧节点的差异：
1.如果vnode不存在，但是oldVnode存在，说明是需要销毁旧节点。
2.如果vnode存在，但是oldVnode不存在，说明是需要创建新节点。
3.当vnode和oldVnode都存在，那就要对节点进行进一步的比较`patchVnode`。

`patchVnode`主要是对节点本身以及节点属性进行比较，对于他们的子节点的比较，就是`updateChildren`

```
function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
      var oldStartIdx = 0;  //旧节点开始索引
      var newStartIdx = 0;  //新节点开始索引
      var oldEndIdx = oldCh.length - 1; //旧节点结束索引
      var oldStartVnode = oldCh[0]; //旧节点开始节点
      var oldEndVnode = oldCh[oldEndIdx];   //旧节点结束节点
      var newEndIdx = newCh.length - 1; //新节点结束索引
      var newStartVnode = newCh[0]; //新节点开始节点
      var newEndVnode = newCh[newEndIdx];   //新节点结束节点
      var oldKeyToIdx, idxInOld, vnodeToMove, refElm;

      // removeOnly is a special flag used only by <transition-group>
      // to ensure removed elements stay in correct relative positions
      // during leaving transitions
      var canMove = !removeOnly;

      {
        checkDuplicateKeys(newCh);
      }

      while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (isUndef(oldStartVnode)) {
          oldStartVnode = oldCh[++oldStartIdx]; // Vnode has been moved left
        } else if (isUndef(oldEndVnode)) {
          oldEndVnode = oldCh[--oldEndIdx];
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
        //如果旧开始节点和新开始节点相同，就递归比较他们两个节点，同时索引往中间递增，递增之后，旧开始节点和新开始节点也发生了变化，变成了原节点的相邻节点
          patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
          oldStartVnode = oldCh[++oldStartIdx];
          newStartVnode = newCh[++newStartIdx];
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
         //如果旧结束节点和新结束节点相同，就递归比较他们两个节点，同时索引往中间递减
          patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
          oldEndVnode = oldCh[--oldEndIdx];
          newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldStartVnode, newEndVnode)) { 
        //如果旧开始节点和新结束节点相同，就递归比较他们两个节点。同时，因为旧开始节点和新结束节点相同，所以把旧开始节点放到旧结束节点的后面。并且变化索引和赋值开始结束节点，继续循环。
          patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
          canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
          oldStartVnode = oldCh[++oldStartIdx];
          newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldEndVnode, newStartVnode)) { 
        //如果旧结束节点和新开始节点相同，就比较他们两个节点。同时，把旧结束节点放到旧开始节点前面。并且变化索引，修改开始结束节点
          patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
          canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
          oldEndVnode = oldCh[--oldEndIdx];
          newStartVnode = newCh[++newStartIdx];
        } else {
        //如果都不满足以上四种情形，那说明没有相同的节点可以复用
          if (isUndef(oldKeyToIdx)) { oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); }
          
          idxInOld = isDef(newStartVnode.key)
            ? oldKeyToIdx[newStartVnode.key]
            : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
            // 把旧节点用map映射的方法，查找是否有新节点与旧节点相同，如果没有找到相同的，就新建。
          if (isUndef(idxInOld)) { // New element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
          } else {
          //如果新老节点相同，就比较他们两个节点，并把新节点放到旧开始节点前边
            vnodeToMove = oldCh[idxInOld];
            if (sameVnode(vnodeToMove, newStartVnode)) {
              patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
              oldCh[idxInOld] = undefined;
              canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
            } else {
              //如果不是相同节点，就新建
              createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
            }
          }
          newStartVnode = newCh[++newStartIdx];
        }
      }
      //如果旧节点列表先循环结束，那么说明新节点列表数量是多于旧节点列表，新开始索引和新结束索引之间的节点就判定为是新增节点，则创建。
      if (oldStartIdx > oldEndIdx) {
        refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
        addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
      } else if (newStartIdx > newEndIdx) {
      //如果新节点列表先循环结束，那么说明旧节点列表数量是多于新节点列表，旧开始索引和旧结束索引之间的节点就判定为是多余节点，则删除。
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
      }
    }
```
以上是diff算法在vue中的应用，整个过程是先进行了：
1.旧开始 《-》新开始
2.旧结束 《-》 新结束
3.旧开始 《-》 新结束
4.旧结束 《-》 新开始
这四个方案的比较。如果这四种都不符合，那就再以旧节点为key进行map映射查找是否有新节点相同的节点，如果有，则把新节点放到旧开始节点前边，如果没有找到，则新建节点。
在整个比较过程中，<b>新旧节点顺序始终是不变的，操作的是真实DOM。</b>
这里推荐一篇文章，非常的通俗易懂-->[链接](https://www.infoq.cn/article/uDLCPKH4iQb0cR5wGY7f)

> computed原码分析

-   原理分析
初始化computed属性是在initState中
```
function initState (vm) {
    ...
    if (opts.computed) { initComputed(vm, opts.computed); }
    ...
  }
```
判断如果有computed属性，就执行initComputed初始化computed
```
function initComputed (vm, computed) {
    //  创建一个空对象，用来保存watchers对象,  在之后获取computed中的key时，会执行getter，getter就会从vm._computedWatchers中找到对应的key的watcher实例
    var watchers = vm._computedWatchers = Object.create(null);
    // 判断是否是服务端渲染
    var isSSR = isServerRendering();

    //  业务代码中声明的computed中的key，对应的方法作为getter
    for (var key in computed) {
      var userDef = computed[key];
      var getter = typeof userDef === 'function' ? userDef : userDef.get;
      if (getter == null) {
        warn(
          ("Getter is missing for computed property \"" + key + "\"."),
          vm
        );
      }

      if (!isSSR) {
        // 创建一个临时watcher实例
        watchers[key] = new Watcher(
          vm,
          getter || noop,
          noop,
          computedWatcherOptions
        );
      }

      //    computed中的key与data,prop不能重复，主要是做判断，如果合法，那就执行defineComputed
      if (!(key in vm)) {
        defineComputed(vm, key, userDef);
      } else {
        if (key in vm.$data) {
          warn(("The computed property \"" + key + "\" is already defined in data."), vm);
        } else if (vm.$options.props && key in vm.$options.props) {
          warn(("The computed property \"" + key + "\" is already defined as a prop."), vm);
        }
      }
    }
  }
```

接下来就是执行defineComputed
```
function defineComputed (
    target,
    key,
    userDef
  ) {
    var shouldCache = !isServerRendering();
    if (typeof userDef === 'function') {
    //  使用Object.defineProperty定义computed中的key,getter就是createComputedGetter(key)
      sharedPropertyDefinition.get = shouldCache
        ? createComputedGetter(key)
        : createGetterInvoker(userDef);
      sharedPropertyDefinition.set = noop;
    } else {
      sharedPropertyDefinition.get = userDef.get
        ? shouldCache && userDef.cache !== false
          ? createComputedGetter(key)
          : createGetterInvoker(userDef.get)
        : noop;
      sharedPropertyDefinition.set = userDef.set || noop;
    }
    if (sharedPropertyDefinition.set === noop) {
      sharedPropertyDefinition.set = function () {
        warn(
          ("Computed property \"" + key + "\" was assigned to but it has no setter."),
          this
        );
      };
    }
    Object.defineProperty(target, key, sharedPropertyDefinition);
  }
```
我们看到该方法是使用Object.defineProperty定义computed中的key,getter就是createComputedGetter(key)，看看他是怎么定义的getter
```
function createComputedGetter (key) {
    return function computedGetter () {
      var watcher = this._computedWatchers && this._computedWatchers[key];
      if (watcher) {
        if (watcher.dirty) {
          watcher.evaluate();
        }
        if (Dep.target) {
          watcher.depend();
        }
        return watcher.value
      }
    }
  }
```
getter的定义是从vm._computedWatchers中找到computed的key的watcher实例，创建watcher实例时，dirty是默认为true。第一次获取key，dirty是为true的话，就通过watcher.evaluate去计算key的属性。
```
 Watcher.prototype.evaluate = function evaluate () {
    this.value = this.get();
    this.dirty = false;
  };
```
这里我们看到就是执行我们自定义的computed的key的方法，得到值赋值给了watcher实例，进行缓存，之后将dirty变为false,这样再次获取computed的key，就直接取值，避免再次计算。
那么再看get()
```
 Watcher.prototype.get = function get () {
    //  将watcher实例赋值给了Dep.target，这里准备做依赖收集了
    pushTarget(this);
    var value;
    var vm = this.vm;
    try {
    //  这里就会去执行我们自定的computed的key的方法，如果方法中用到了data中的响应式属性，就会触发该属性的getter
      value = this.getter.call(vm, vm);
    } catch (e) {
      if (this.user) {
        handleError(e, vm, ("getter for watcher \"" + (this.expression) + "\""));
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value);
      }
      popTarget();
      this.cleanupDeps();
    }
    return value
  };
```
这里我们看到去执行我们自定的computed的key的方法，如果方法中用到了data中的响应式属性，就会触发data的属性的getter
```
get: function reactiveGetter () {
    var value = getter ? getter.call(obj) : val;
    //  此时的Dep.target就是computed的key的watcher实例，就会进行依赖收集
    if (Dep.target) {
      dep.depend();
      if (childOb) {
        childOb.dep.depend();
        if (Array.isArray(value)) {
          dependArray(value);
        }
      }
    }
    return value
  },
```
这样一来，当computed的key所依赖的data的key发生变化时，就会触发data的key的dep.notify()
```
  Dep.prototype.notify = function notify () {
    // stabilize the subscriber list first
    var subs = this.subs.slice();
    if (!config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort(function (a, b) { return a.id - b.id; });
    }
    for (var i = 0, l = subs.length; i < l; i++) {
      subs[i].update();
    }
  };
```
但是在update()中，并不会立即重新计算computed的key的值，而是把它的dirty变为true
```
/**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  Watcher.prototype.update = function update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true;
    } else if (this.sync) {
      this.run();
    } else {
      queueWatcher(this);
    }
  };
```
这是因为，当data的属性发生变化，就会重新渲染视图，触发_render重新渲染，而视图上依赖了computed的key,就会去获取这个key,触发key的getter,在getter中就会由于dirty为true，而再次evaluate()。
这便是整个computed的处理过程。

> watch源码分析

- 原理分析
初始化watch的入口是在initState中
```
function initState (vm) {
    ...
    if (opts.watch && opts.watch !== nativeWatch) {
      initWatch(vm, opts.watch);
    }
  }
```
看一下initWatch
```
function initWatch (vm, watch) {
    for (var key in watch) {
      var handler = watch[key];
      if (Array.isArray(handler)) {
        for (var i = 0; i < handler.length; i++) {
          createWatcher(vm, key, handler[i]);
        }
      } else {
        createWatcher(vm, key, handler);
      }
    }
  }
```
根据观察的key的操作是否是数组还是单个函数，进行createWatcher操作
```
function createWatcher (
    vm,
    expOrFn,
    handler,
    options
  ) {
    ...
    //  这里的传参就是观察的key,自定义观察的key的函数
    return vm.$watch(expOrFn, handler, options)
  }
  
```
返回的vm.$watch就是要去创建一个watcher实例

```
Vue.prototype.$watch = function (
      expOrFn,
      cb,
      options
    ) {
      ...
      var watcher = new Watcher(vm, expOrFn, cb, options);
      ...
    };
  }
```

```
var Watcher = function Watcher (
    vm,
    expOrFn,
    cb,
    options,
    isRenderWatcher
  ) {
    ...
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn;
    } else {
      this.getter = parsePath(expOrFn);
      if (!this.getter) {
        this.getter = noop;
        warn(
          "Failed watching path: \"" + expOrFn + "\" " +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        );
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get();
  };
```
在这里会去通过执行this.get()去获取watcher实例的value，这便是准备开始收集依赖了，和computed类似。
```
Watcher.prototype.get = function get () {
    //这里会把Dep.target设为watcher实例
    pushTarget(this);
    var value;
    var vm = this.vm;
    try {
        //然后会去执行观察的data的key的getter
      value = this.getter.call(vm, vm);
    } catch (e) {
      if (this.user) {
        handleError(e, vm, ("getter for watcher \"" + (this.expression) + "\""));
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value);
      }
      popTarget();
      this.cleanupDeps();
    }
    return value
  };
```
在这里我们只需要关心先把全局的Dep.target设为watcher实例，然后就去执行Object.defineProperty的getter,在getter中会判断如果有Dep.target,就会进行依赖收集.
```
Object.defineProperty(obj, key, {
      enumerable: true,
      configurable: true,
      get: function reactiveGetter () {
        var value = getter ? getter.call(obj) : val;
        //  此时的target是上面提到的创建的watcher实例
        if (Dep.target) {
            //进行依赖收集
          dep.depend();
          if (childOb) {
            childOb.dep.depend();
            if (Array.isArray(value)) {
              dependArray(value);
            }
          }
        }
        return value
      },
      ...
  }
```
这样watch的key就完成了依赖的收集，当data的响应式属性变化时，就会dep.notify()通知subs中的watcher实例进行更新。
```
Watcher.prototype.update = function update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true;
    } else if (this.sync) {
      this.run();
    } else {
      queueWatcher(this);
    }
  };
```
更新并不是立即更新，而是通过queueWatcher放到队列中，再通过nextTick执行。
以上，便是watch的源码分析。

> vue-router源码分析

-   介绍
vue-router是官方的前端路由库，作为一个单页应用，路由功能可以使单页应用看起来更像个多页应用，可以更好地控制页面跳转。Vue提供的路由方式有`hash`,`history`,`abstract`。本文以v2.8为准来介绍。

-   原理分析
Vue引入插件是通过`Vue.use(plugin)`,`use`源码是这样定义的：
```
export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```
主要是判断避免重复引入插件。初始化插件是把插件的`install`或插件导出的定义在此上下文中执行。

`install`的代码如下：
```
export function install (Vue) {
  if (install.installed && _Vue === Vue) return
  install.installed = true

  _Vue = Vue

  const isDef = v => v !== undefined

  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }

  Vue.mixin({
    beforeCreate () {
      if (isDef(this.$options.router)) {
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })

  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })

  Vue.component('router-view', View)
  Vue.component('router-link', Link)

  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}
```

该方法使每个组件混入`beforeCreate`和`destroyed`方法,并响应式声明`_route`属性，定义`router-view`,`router-link`组件，并定义组件生命周期钩子。

`install`是定义在`VueRouter`上的一个方法，回看`VueRouter`类是怎么声明的：
```
    ...
    this.matcher = createMatcher(options.routes || [], this)
    //默认是hash模式，如果浏览器不支持history,就选择hash模式
    let mode = options.mode || 'hash'
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    if (this.fallback) {
      mode = 'hash'
    }
    //如果不是在浏览器，就选择abstract模式
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode

    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this, options.base)
        break
      default:
        if (process.env.NODE_ENV !== 'production') {
          assert(false, `invalid mode: ${mode}`)
        }
    }
    ...
```
通过`createMatcher`生成路由映射信息，然后对路由模式优雅降级,然后根据模式，选择具体的路由实体类。

router跳转是通过push方法，那么来看看push是怎么定义的：
```
push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    //先执行transitionTo,并传了个回调成功函数
    this.transitionTo(location, route => {
      pushState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }
```

```
transitionTo (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const route = this.router.match(location, this.current)
    //先确认是否跳转
    this.confirmTransition(route, () => {
      this.updateRoute(route)
      onComplete && onComplete(route)
      this.ensureURL()

      // fire ready cbs once
      if (!this.ready) {
        this.ready = true
        this.readyCbs.forEach(cb => { cb(route) })
      }
    }, err => {
      if (onAbort) {
        onAbort(err)
      }
      if (err && !this.ready) {
        this.ready = true
        this.readyErrorCbs.forEach(cb => { cb(err) })
      }
    })
  }
```
`confirmTransition`主要是判断是否是同一路由之间的跳转，如果不是的话，就把跳转之后需要调的任务放到任务队列中，这些任务主要是些生命周期钩子。
确认跳转之后就通过`updateRoute`更新路由，然后再执行push中的回调`pushState`，
具体看下updateRoute:
```
updateRoute (route: Route) {
    const prev = this.current
    this.current = route
    this.cb && this.cb(route)
    this.router.afterHooks.forEach(hook => {
      hook && hook(route, prev)
    })
  }
```
这里执行cb,那么cb是在哪定义的呢？其实是在init中：
```
history.listen(route => {
    this.apps.forEach((app) => {
        app._route = route
    })
})
```
又因为在前面定义了响应式属性`_route`：
```
Vue.util.defineReactive(this, '_route', this._router.history.current),
```
所以就会触发组件的setter, setter中调用dep中收集的依赖，触发render,再通过nextTick重新渲染。
router-view中组件定义了render方法，在该方法中其实是执行$createElement去渲染组件

> Vuex原理分析

-   介绍
Vuex是一个可以全局组件共享数据的插件，通过actions -> mutations -> state的流程，可以更加规范化数据操作。

-   原理分析
Vue通过`install`方法加载Vuex实例，主要是通过`mixin`在`beforeCreate`时来给Vue扩展方法：
```
Vue.mixin({ beforeCreate: vuexInit })
```

```
 function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
```
`vuexInit`主要是把store实例挂载到this.$store上，子组件从其父组件引用$store属性，层层嵌套进行设置,以此来达到给全局组件注入store的目的。
那么再来看Store是怎么定义的：
```
export class Store {
    constructor (options = {}) {
        //省略部分代码
        ...
        
        //当正在执行mutations时，就改变_committing状态，改变结束，再更改回来，用于区分是否是通过mutations来改变state,而不是直接修改state
        this._committing = false
        
        //把定义的action方法放进对象做一个映射集
        this._actions = Object.create(null)
        
        //把定义的mutation方法放进对象做一个映射集
        this._mutations = Object.create(null)
     
    
        // 定义dispatch和commit
        const store = this
        const { dispatch, commit } = this
        this.dispatch = function boundDispatch (type, payload) {
          return dispatch.call(store, type, payload)
        }
        this.commit = function boundCommit (type, payload, options) {
          return commit.call(store, type, payload, options)
        }
    
        //初始化data getters
        resetStoreVM(this, state)
        
        // 安装插件
        plugins.forEach(plugin => plugin(this))
    }
}
```
实例化Store中，把定义的mutation和action都各自放进了一个映射集合，然后在执行时，通过方法名称去执行对应方法，然后再看是如何定义了dispatch和commit的：
```
dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    
    //通过type找到需要执行的方法
    const entry = this._actions[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown action type: ${type}`)
      }
      return
    }

    this._actionSubscribers.forEach(sub => sub(action, this.state))
    
    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }

```
`commit`和`dispatch`类似，只不过多了一句
```
 this._withCommit(() => {
  entry.forEach(function commitIterator (handler) {
    handler(payload)
  })
})
```
这个_withCommit就是用来在mutation改变state时，改变一下状态：
```
_withCommit (fn) {
    const committing = this._committing
    this._committing = true
    fn()
    this._committing = committing
}
```
再看state和getter是怎么定义的,这两个的定义是在初始化Store时，有个
```
resetStoreVM(this, state)
```
```
function resetStoreVM (store, state, hot) {
    //省略部分代码
    ...
    
    ...
    const computed = {}
      forEachValue(wrappedGetters, (fn, key) => {
        // use computed to leverage its lazy-caching mechanism
        computed[key] = () => fn(store)
        Object.defineProperty(store.getters, key, {
          get: () => store._vm[key],
          enumerable: true // for local getters
        })
      })
    ...
    
    store._vm = new Vue({
        data: {
          $$state: state
        },
        computed
      })
      
    ...
}
```
可以看到，通过创建一个响应式的data和computed来实现state和getters。
我们在在组件中使用Vuex时，经常会通过`import { mapGetters, mapActions } from 'vuex'`来直接使用action或mutation的方法，那么他们是怎么实现的呢：
```
export const mapActions = normalizeNamespace((namespace, actions) => {
  const res = {}
  normalizeMap(actions).forEach(({ key, val }) => {
    res[key] = function mappedAction (...args) {
      let dispatch = this.$store.dispatch
      //如果有命名空间
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapActions', namespace)
        if (!module) {
          return
        }
        dispatch = module.context.dispatch
      }
      return typeof val === 'function'
        ? val.apply(this, [dispatch].concat(args))
        : dispatch.apply(this.$store, [val].concat(args))
    }
  })
  return res
})
```
其实就是从_actions方法集合中,找到该方法，dispatch绑定$store上下文，并返回新的方法集合。