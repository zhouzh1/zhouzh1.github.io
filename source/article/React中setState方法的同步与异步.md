---
title: React中setState方法的同步与异步
tags: [React]
category: Frontend
filename: sync and async of react setState
createdTime: 2019-10-24 17:41
---
### this.setState方法内部做了什么？
调用this.setState方法后，先进行状态合并，并会触发状态更新，重新调用组件的render方法，render方法执行结束后会产生新的虚拟dom，最后React内部会对组件的新旧虚拟dom进行diff操作，得到针对旧虚拟dom的patch，最后根据patch来更新dom结构，实现视图更新.

### this.setState的同步和异步，为什么这样处理？
React的setState方法本质上都是同步操作，所谓异步，指的是将一系列的setState操作先收集起来，最终在函数执行结束时进行状态合并后再进行批量执行.

之所以这样做，是因为如前文所述，每次调用setState方法都会触发视图更新，先将多次setState的调用收集起来，最终根据每次setState调用的状态参数进行状态合并后会得到最终的组件状态，这样一来，虽然调用了多次this.setState，但最终只会调用一次组件的render方法，视图也只会更新一次.

### 同步和异步分别发生在什么时机？
如前文所述，在函数的执行过程中，如果React想对this.setState进行“异步”操作，必须要能知晓这个函数什么时候会执行结束，在React中，组件生命周期函数的执行完全由React控制，事件处理程序由于合成事件的机制也被React所控制，因此在这两类函数里的this.setState都是“异步”处理，对于setTimeout和setInterval以及Promise回调这一类不在React管控范围内的函数里面，所有的this.setState就只能是同步处理了.

```js
import React from 'react';

class Foo extends React.Component {

    state = {
        a: 'a',
    }

    bar() {
        this.setState({
            a: 'A'
        });
        console.log(this.state.a); // 'a'
    }

    componentDidMount() {
        setTimeout(() => {
            this.setState({
                a: 'A'
            });
            console.log(this.state.a); // 'A'
        }, 0);

        Promise.resolve().then(() => {
            this.setState({
                a: 'A'
            });
            console.log(this.state.a); // 'A'
        });

        this.setState({
            a: 'A'
        });
        console.log(this.state.a); // 'a'
    }

    render() {
        return (
            <div onClick={this.bar}>bar</div>
        )
    }
}
```