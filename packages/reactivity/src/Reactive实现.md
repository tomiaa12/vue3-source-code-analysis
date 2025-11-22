# Reactive 设计

## Proxy 代理

```ts
const mutableHandlers: ProxyHandler<any> = {
  get(target, key, recevier) {},
  set(target, key, value, recevier) {},
};

function createReactiveObject(target) {
  let proxy = new Proxy(target, mutableHandlers);
  return proxy;
}
```

## 重复代理

```js
// 一个对象重复代理时
const obj = {};
const state = reactive(obj);
const state1 = reactive(obj);

console.log(state === state1);

// 实现如下
// 用于记录我们的 代理后的结果，可以复用
const reactiveMap = new WeakMap();
function createReactiveObject(target) {
  // 统一做判断，响应式对象必须是对象才可以
  if (!isObject(target)) return target;

  // 取缓存，如果有直接返回
  const exitsProxy = reactiveMap.get(target);
  if (exitsProxy) return exitsProxy;

  let proxy = new Proxy(target, mutableHandlers);
  // 根据对象缓存 代理后的结果
  reactiveMap.set(target, proxy);
  return proxy;
}
```

## 传入已经被代理的 Proxy 实例

```ts
// 传入已经被代理过的对象时
const state = reactive({});
const state1 = reactive(state);

console.log(state === state1);

// 实现如下
enum ReactiveFlags {
  IS_REACTIVE = "__v_isReactive", // 基本上唯一
}
const mutableHandlers: ProxyHandler<any> = {
  get(target, key, recevier) {
    // 如果触发了 ReactiveFlags.IS_REACTIVE 则直接返回 true
    if (key === ReactiveFlags.IS_REACTIVE) {
      return true;
    }
    // do xxxx
  },
  set(target, key, value, recevier) {},
};

function createReactiveObject(target) {
  // 如果已经被 Proxy代理，取值 target[ReactiveFlags.IS_REACTIVE] 会触发 get 方法
  if (target[ReactiveFlags.IS_REACTIVE]) {
    return target;
  }

  // 其他代码...
  let proxy = new Proxy(target, mutableHandlers);
  return proxy;
}
```

## get 取值问题

```ts
const person = {
  name: "jw",
  get aliasName() {
    return this.name + "handsome";
  },
};

let proxyPerson = new Proxy(person, {
  // target 是
  // recevier 是代理对象
  get(target, key, recevier) {
    // 1. 如果是这样返回 return target[key]
    // console.log(key);
    // log ==> aliasName
    // 只会打印 aliasName
    // 但是 aliasName 又取了 this.name，取 name 的时候没有触发 proxy 的 get
    // return target[key];

    // 2. 如果用recevier 取 key 会导致死循环 无限触发 get
    // return recevier[key];

    // 3. 解决：使用 Reflect 来获取
    console.log(key);
    // aliasName
    // name
    // Reflect 读取aliasName内部，this.name 指向了 Proxy，所以也会触发 get方法打印 name，
    return Reflect.get(target, key, recevier);  // person.name 不会触发get
  },
});

console.log(proxyPerson.aliasName);
```
