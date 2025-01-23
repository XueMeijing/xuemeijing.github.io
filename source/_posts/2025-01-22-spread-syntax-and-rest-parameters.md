---
title: 深入理解展开语法和剩余参数
date: 2025-01-22 11:19:03
categories: 
- 有趣
---

写文档之前我首先要吐槽一下，真是不吐不快。最近半年被薅去搞我们公司sass产品的定制化开发去了，期间有些需求是外包做的，两个月接触下来终于理解了为什么hr会那么歧视有外包工作经验的人了，简历个个都是五年八年，一千多一人天的，结果一问三不知，还不如实习生，实习生起码愿意学。

- 连代码都找不到在哪，用react devTools找不到在哪，就不会全局搜关键字吗，代码里搜不到就网络接口里搜啊，干什么都要react devTools，Elements是满足不了你是吧
![Image](https://github.com/user-attachments/assets/a0936780-e6a9-470d-8bf8-6ec1ed19d094)

- 连最基础的git都用不明白，git stash 没听过，说了也不用，代码回撤靠手动一行一行删，ssh生成密钥不会也不知道百度（虽然百度很垃圾，但是遇到问题连百度都不会，就知道问问问的人不是更垃圾吗？）发了《提问的智慧》国内链接，估计也是没看过，搞得我把签名都改成好消息，好消息，本人已与百度达成战略合作，以后不会的可以直接问百度了
![Image](https://github.com/user-attachments/assets/c4086979-e5d2-442b-a771-6e52b342ca8f)

- 一个前端开发用webstorm就算了，咱也不是有歧视的人，你就是用文本编辑器写我也没意见，主要是你就不能管管你的编辑器吗？老是自动去行尾空格，review 代码的时候，明明只有一行改动，我要对比几十行，说了几次都不改，也不知道手动百度一下
![Image](https://github.com/user-attachments/assets/ddf19b2a-46b3-44d8-abbf-54282a8e1132)

- 让对齐一下html标签，回复理由竟然是怕手动对齐不准确，一时间我竟无言以对
![Image](https://github.com/user-attachments/assets/a63befb7-00f5-4a24-9cc3-c10d2641b4ba)
![Image](https://github.com/user-attachments/assets/896388e2-6c23-4504-bd79-75310b9de387)

- 参数三个不行，两个就正常，也不看参数传到哪里，给谁用了
![Image](https://github.com/user-attachments/assets/14c55c0b-ba1d-4200-b35a-6eb9ce92c3c7)

- 前面明明有判断，是否存在某个变量，把这条语句删了直接对变量解构赋值了
![Image](https://github.com/user-attachments/assets/1d958d24-5ea7-47e1-974a-11a3e0e51afa)

- 把数组变量当作或运算的条件，就算是空数组也是true啊
![Image](https://github.com/user-attachments/assets/1d6e0f16-f16e-4c22-8380-7bf0bcefb1aa)

还有其他各种问题，就懒的一一列举了，我个人感觉我已经很有耐心了，也教了他很多，但是上面的这些问题都不是技术能力导致的，而是学习能力和态度问题，写代码不动脑子，写完不检查就提交，也不让gpt检查下自己写的有没有问题

## 展开语法

为什么要吐槽上面一堆呢，因为他还写了一段下面的代码
```javascript
value.push({
  ...(variableA === 'A' && {
    a,
  })
});
```
我看到真的气笑了，这是人能写出来的啊？variableA === 'A'为true的时候正常，为false的时候，展开运算符也能展开布尔值吗？嗨你别说还真能，这回让他瞎猫碰上死耗子了

![Image](https://github.com/user-attachments/assets/65b2f50d-4eab-4079-82da-a95ec17eb838)

为什么会出现这种情况呢，[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax) 上不是说只能展开数组、字符串和字面量对象吗？

cursor 告诉我，对象展开运算符 (...) 在对象字面量中的行为和在数组中的行为是不同的：
数组展开运算符：确实只能展开可迭代对象（iterables），比如数组、字符串、Set、Map等
```javascript
// 正确
[..."abc"]  // ["a", "b", "c"]
[...["a", "b"]]  // ["a", "b"]
```

```javascript
// 错误 - 会抛出错误
[...undefined]  // TypeError: undefined is not iterable
[...42]  // TypeError: 42 is not iterable
```

对象展开运算符：可以展开任何值，但只有对象类型会产生实际效果
```javascript
// 对象展开的行为：
const obj = {
  ...undefined,  // 不会添加任何属性
  ...null,       // 不会添加任何属性
  ...42,         // 不会添加任何属性
  ...true,       // 不会添加任何属性
  ...{a: 1}      // 会添加 a: 1
};
console.log(obj);  // {a: 1}
```

为什么会有这种情况呢？他说了一下简化版的对象展开运算符内部实现原理，但是我没找到对应的源代码，v8都是c写的找不到

```javascript
// 简化版的对象展开运算符内部实现原理
function objectSpread(target, source) {
  // 1. 将 source 转换为对象
  const fromObj = Object(source);
  
  // 2. 获取所有可枚举属性
  const keys = Object.keys(fromObj);
  
  // 3. 复制属性
  for (const key of keys) {
    target[key] = fromObj[key];
  }
  
  return target;
}

// 使用示例
const result = {
  ...undefined,  // Object(undefined) 返回空对象 {}
  ...null,       // Object(null) 返回空对象 {}
  ...42,         // Object(42) 返回 Number 对象，但没有自身可枚举属性
  ...{a: 1}      // 正常复制属性
};
```

但是我测试下来发现布尔值是个例外，Object(false)会返回Boolean {false}，Boolean {false} === false 又为true，这不是又回到...false死循环了吗，不知道怎么做的。

这种设计的原因主要有：
- 容错性考虑
- 允许展开非对象值而不抛出错误，使代码更健壮
- 特别是在处理可能为 undefined 的动态数据时很有用
- 一致性考虑
- 与 Object() 构造函数的行为保持一致
- 与 JavaScript 其他隐式类型转换的宽松特性保持一致

为什么数组不采用这种容错性考虑呢？因为数组是有序集合,需要明确的迭代规则来保证顺序，所以他使用了迭代器
1. 获取迭代器
- 通过 GetIterator() 抽象操作获取被展开对象的迭代器
- 如果对象不可迭代,会抛出 TypeError
2. 迭代过程
- 通过迭代器的 next() 方法获取每个值
- 将获取到的值按顺序插入到数组中

## 剩余参数

我看了MDN才知道还有剩余参数这个说法, 虽然我一直使用他们，但是没考虑过它们之间细微的区别

首先，展开语法和剩余参数的符号都是三个点 ... , 但是它们的作用是相反的
- 展开语法：将一个数组或可迭代对象展开为多个独立的值
- 剩余参数：将多个独立的值组合成一个数组

剩余参数主要用于接受函数参数和解构赋值中，类似arguments，但是arguments是类数组，而剩余参数是真正的数组

用在函数参数中
```javascript
function sum(...args) {
  return args.reduce((a, b) => a + b, 0);
}
console.log(sum(1, 2, 3, 4));  // 10
```

用在解构赋值中
```javascript
const [first, ...rest] = [1, 2, 3, 4];
console.log(first);  // 1
console.log(rest);   // [2, 3, 4]
```

当然剩余参数也可以用在对象解构中，只是不知道为什么MDN上没有提到
```javascript
const { a, ...rest } = { a: 1, b: 2, c: 3 };
console.log(a);  // 1
console.log(rest);  // { b: 2, c: 3 }
```

给MDN提issue了，希望他们能更新一下 https://github.com/mdn/mdn/issues/629
