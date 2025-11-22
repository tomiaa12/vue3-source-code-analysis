# Effect 实现

```ts
class ReactiveEffect {
  constructor(fn, scheduler) {
    this.fn = fn;
    this.scheduler = scheduler;
  }

  run() {
    this.fn();
  }
}

function effect(fn) {
  // 当依赖数据 fn 变化，就会执行 run
  const _effect = new ReactiveEffect(fn, () => {
    _effect.run();
  });

  // 创建完成后，执行一次
  _effect.run();

  return _effect;
}

// 示例
const obj = {
  name: "tomiaa",
  age: 18,
};
const _effect = effect(() => {
  app.innerHTML = `name: ${obj.name} age: ${obj.age}`;
});

obj.age++;

// 数据变了，自动执行 run，这里手动执行 run 演示
_effect.run();
```
