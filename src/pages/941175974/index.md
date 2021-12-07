---
title: '设计模式实际应用系列-策略模式'
date: '2021-07-10'
spoiler: ''
---

  相信很多同学在做表单验证时，少不了一些规则的校验，比如下列代码：

```jsx
//模拟表单验证
document.getElementById('formId').onclick = function() {
  //伪代码
  if (!nameInput.value) {
    toast('输入框内容不能为空')
    return
  }
  if (pwdInput.valuee < 6) {
    toast('密码不能小于6位数')
    return
  }
  if (!/(^1[0-9]{9}$)/.test(phone)) {
    toast('手机格式不正确')
    return
  }
  //onSumbit
}
```

如果项目中有多个表单校验，相信也有不少同学为了省事，就直接复制粘贴了。这样项目中充斥着相同的代码，不能复用，而且后面再加入其他的校验规则，这里的代码只会越来越多。

如果某有一个需求变更，需要对校验规则做修改，也就只能去修改表单验证的逻辑部分。那么其他的表单是不是也要一块修改呢？所以这样做着重复性的工作是毫无意义的。

为了解决这个问题，可以采用策略模式，将通用的规则校验封装成一个策略对象。即使是修改规则，只需要修改这个策略对象即可。

```jsx

//维护一个所有规则对象
const strategies = {
  isValueEmpty: function(value, errorMsg) {
    if (value === '') {
      return errorMsg
    }
  },
  minLength: function(value, minLength, errorMsg) {
    if (value.length < minLength) {
      return errorMsg
    }
  },
  isPhone: function (phone, errorMsg) {
    if(!(/^1[3456789]\d{9}$/.test(phone))){ 
      return errorMsg
    }
  },
  isSpecialText: function(value, errorMsg) {
    const regEn = /[`!@#$%^&*()_+<>?:"{},.\/;'[\]]/im,
    regCn = /[·！#￥（——）：；“”‘、，|《。》？、【】[\]]/im;
    if (regEn.test(value) || regCn.test(value)) {
      return errorMsg
    }
  }
}
......

const validator = {
  cache: [], //存储校验规则
  add: function(value, rule, errorMsg) {
    let _array = rule.split(':')
    const strategy = _array.shift()
    _array.unshift(value)
    _array.push(errorMsg)
    this.cache.push(function() {
      return strategies[strategy].apply(value, _array)
    })
  },
  valid: function() {
    for (let index = 0; index < this.cache.length; index++) {
      const validatorFunc = this.cache[index]
      const errorMsg = validatorFunc()
      if (errorMsg) { //如果有错误信息返回，立即中断循环，返回该错误信息
        return errorMsg
      }
    }
  }
}


```
那么针对每一个每一个表单验证，或者多个表单验证，我们需要写一个验证函数。

```jsx
const validatorFun = function() {
  const name = 'js'
  const pwd = '123456'
  const phone = '13011112222'
  const text = '$${{}{[]][]【】】'
  validator.add(name, 'isValueEmpty', '姓名不能为空')
  validator.add(pwd, 'minLength:6', '密码不能小于6位数')
  validator.add(phone, 'isPhone', '手机号码格式不对')
  validator.add(text, 'isSpecialText', '不能输入特殊字符串')
  return validator.valid()
}
```

那么在表单校验中，如下执行：

```jsx
document.getElementById('formId').onclick = function() {
  const errorMsg = validatorFun()
  if (errorMsg) {
    alert(errorMsg)
    return
  }
}

```

将不同的执行逻辑、算法封装到一个通用的策略对象中，可以大大提高代码的复用性。避免了很多种重复条件筛选。因此我们只需要去重点关心算法、执行逻辑。而不是复制粘贴其他表单验证的代码。
即使后面面对不同的需求场景，只需修改策略算法核心层面，就能轻松应对变化的需求场景。 这也是策略模式的优点所在。

  