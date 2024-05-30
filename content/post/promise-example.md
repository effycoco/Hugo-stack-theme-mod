---
title: "Promise示例"
description: 
date: 2024-05-30T15:40:55+08:00
image: 
categories:
slug:
tags:
math: 
license: 
hidden: false
comments: true
draft: false
---
术语解释：
- Promise 对象的结果这个结果可以是成功的值（resolved value）或拒绝的原因（rejection reason）。
- 对 Promise 对象添加回调函数：例如 `this.$post().then(res=>{})` 就是通过 `.then()` 注册了回调函数。
- 如果 `Promise` 对象状态已经改变，你再对它添加回调函数，也会立即得到这个结果。意思是不管立即添加回调函数还是稍后添加，都能获取结果。
```js
axios.post().then(res=>{}); // 这样是立即添加回调函数

// 这样是过段时间再添加回调函数
const request = axios.post();
// 执行其他代码或等待一段时间
request.then(res=>{})
```




# 示例

需求
- 用户可以选择常规物品列表中的物品进行下单，也可以输入其他物品进行下单。
- 如果只有其他物品，需要先调用 `goods/addApplyGoods` 接口添加其他物品，然后再调用 `goods/apply` 接口下单。
- 如果只有常规物品，直接调用 `goods/apply` 接口下单。
- 如果既有常规物品又有其他物品，需要先调用 `goods/addApplyGoods` 接口添加其他物品并获取物品ID，然后再调用 `goods/apply` 接口下单，常规物品和其他物品分开下单。


## Async 方案 （推荐）
```js
// vue的methods中
async dialogClick(){
    let loading = this.$loading({
        lock: true,
        spinner: "el-icon-loading",
        background: "rgba(0, 0, 0, 0.2)",
    });
    loading.$el.lastElementChild.style.fontSize = "40px";
    try{
        if(!this.submitOrderList.length){ // 只有其他物品
            await this.addOtherGoods();
            await this.applyOtherGoods();
        }else if(!this.otherList.length){ // 只有常规物品
            await this.applyOrdinaryGoods();
        }else{ // 两者都有（进入此函数前已排除两者都无的情况）
            await this.addOtherGoods();
            await Promise.all([this.applyOtherGoods(), this.applyOrdinaryGoods()]);
        }
        loading.close();
        this.$router.push({ path: "/ApplicationCompleted" });
    } catch(error){
        console.error(error);
        loading.close();
        this.$store.commit("comment/UP_ERRORINFO", {
            type: "fail",
            msg: error.message,
            timestamp: new Date().getTime()
        });
    }        
},
addOtherGoods(){
    let otherGoods = this.otherList.map(item=>{
        return {gNAME:item.name};
    });
    let params = {
        token: this.$store.state.tab.token,
        goods: JSON.stringify(otherGoods)
    };
    return this.$post(`goods/addApplyGoods`,params).then(res=>{
        let requestData = this.$NodeRSA.decryptByPrivateKey(
            res.requestData,
            res.encrypted
        );
        this.otherList.forEach((item, index) => {
            item.id = requestData.data[index].gID;
        });
    });
},
applyOtherGoods(){
    let otherFinal = this.otherList.map(item => {
        return {
            gNAME: item.name,
            gUSENUM: item.num,
            gID: item.id
        };
    });
    let params = {
        token: this.$store.state.tab.token,
        goods: JSON.stringify(otherFinal)
    };
    return this.$post(`goods/apply`, params).then(res=>{
        let requestData = this.$NodeRSA.decryptByPrivateKey(
            res.requestData,
            res.encrypted
        );
        console.log(requestData, 'other goods submit order result');
        if (requestData.code !== 0) {
            throw new Error(requestData.message);
        }
    });
},
applyOrdinaryGoods() {
    let params = {
        token: this.$store.state.tab.token,
        goods: JSON.stringify(this.submitOrderList)
    };
    return this.$post(`goods/apply`, params)
    .then(res => {
        let requestData = this.$NodeRSA.decryptByPrivateKey(
            res.requestData,
            res.encrypted
        );
        console.log(requestData, 'ordinary goods submit order result');
        if (requestData.code !== 0) {
        throw new Error(requestData.message);
        }
    });
},
```

后三个方法还可进一步改为async await的形式：
```js
async addOtherGoods() {
  let otherGoods = this.otherList.map(item => {
    return { gNAME: item.name };
  });

  let params = {
    token: this.$store.state.tab.token,
    goods: JSON.stringify(otherGoods)
  };

  const res = await this.$post(`goods/addApplyGoods`, params);

  let requestData = this.$NodeRSA.decryptByPrivateKey(
    res.requestData,
    res.encrypted
  );

  this.otherList.forEach((item, index) => {
    item.id = requestData.data[index].gID;
  });
}

async applyOtherGoods() {
  let otherFinal = this.otherList.map(item => {
    return {
      gNAME: item.name,
      gUSENUM: item.num,
      gID: item.id
    };
  });

  let params = {
    token: this.$store.state.tab.token,
    goods: JSON.stringify(otherFinal)
  };

  const res = await this.$post(`goods/apply`, params);

  let requestData = this.$NodeRSA.decryptByPrivateKey(
    res.requestData,
    res.encrypted
  );

  console.log(requestData, 'other goods submit order result');

  if (requestData.code !== 0) {
    throw new Error(requestData.message);
  }
}

async applyOrdinaryGoods() {
  let params = {
    token: this.$store.state.tab.token,
    goods: JSON.stringify(this.submitOrderList)
  };

  const res = await this.$post(`goods/apply`, params);

  let requestData = this.$NodeRSA.decryptByPrivateKey(
    res.requestData,
    res.encrypted
  );

  console.log(requestData, 'ordinary goods submit order result');

  if (requestData.code !== 0) {
    throw new Error(requestData.message);
  }
}
```

## 链式调用方案
```js
methods: {
  dialogClick() {
    let loading = this.$loading();

    this.addOtherGoods()
      .then(() => {
        return Promise.all([this.applyOtherGoods(), this.applyOrdinaryGoods()]);
      })
      .then(() => {
        loading.close();
        this.$router.push({ path: "/ApplicationCompleted" });
      })
      .catch(error => {
        console.error(error);
        loading.close();
        this.$store.commit("comment/UP_ERRORINFO", {
          type: "fail",
          msg: error.message,
        });
      });
  },
  addOtherGoods() {
    let params = { /* ... */ };
    return this.$post(`goods/addApplyGoods`, params).then(res => {
      let response = this.$NodeRSA.descryptByPrivateKey(res.requestData, res.encrypted);
      this.otherList.forEach((item, index) => { item.id = requestData.data[index].gID; });
    });
  },
  applyOtherGoods() {
    let otherFinal = this.otherList.map(item => { /* ... */ });
    let params = { goods: JSON.stringify(otherFinal) };
    return this.$post(`goods/apply`, params).then(res => {
      let response = this.$NodeRSA.descryptByPrivateKey(res.requestData, res.encrypted);
      if (requestData.code !== 0) {
        throw new Error(requestData.message);
      }
    });
  },
  applyOrdinaryGoods() {
    let params = { goods: JSON.stringify(this.submitOrderList) };
    return this.$post(`goods/apply`, params).then(res => {
      let response = this.$NodeRSA.descryptByPrivateKey(res.requestData, res.encrypted);
      if (requestData.code !== 0) {
        throw new Error(requestData.message);
      }
    });
  }
}
```

需知道以下代码是等价的：

```js
await this.addOtherGoods();
await Promise.all([this.applyOtherGoods(), this.applyOrdinaryGoods()]);
```

```js
this.addOtherGoods()
    .then(() => {
        return Promise.all([this.applyOtherGoods(), this.applyOrdinaryGoods()]);
    })
```

```js
let params = { /* ... */ };
this.$post(`goods/addApplyGoods`, params).then(res => {
  let response = this.$NodeRSA.descryptByPrivateKey(res.requestData, res.encrypted);
  this.otherList.forEach((item, index) => { item.id = requestData.data[index].gID; });
  return Promise.all([this.applyOtherGoods(), this.applyOrdinaryGoods()]);
});
```

若想提取代码到单独的函数，可以将一个回调函数拆成两个，一个放在函数定义的返回语句，一个放在函数调用部分。

当遇到 `await` 关键字时，代码会等待当前 `await` 表达式的 Promise 对象被解析后再继续执行下一行代码。这样可以保证在发送下一个请求之前，前一个请求已经获得了响应。

