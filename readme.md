```javascript
// 响应式更新, 晚饭+消费=总计
// finished.
let watchingFunction = null
function observe(data) {
    const depends = {}
    window.depends = depends // 为了方便浏览器控制台测试
    return new Proxy(data, {
        get: function (obj, key) {
            // 依赖收集
            console.log('getter', obj, key)
            if (watchingFunction){
                if (!depends[key]){
                    depends[key] = []
                }
                depends[key].push(watchingFunction)
            }
            return obj[key]
        },
        set: function (obj, key, value) {
            // 触发依赖 --> 更新
            obj[key] = value
            if (depends[key]) {
                depends[key].forEach(callback=>callback())
            }
        }
    })
}
let data = observe({
    dinnerPrice: 100,
    tip: 10,
    total: 0
})
window.data = data;
// ------------------------------------------------------------------------------
function watcher(target) {
    watchingFunction = target // 当前31行, watchingFunction不为null
    target() // 当前32行, 执行target() 函数
    watchingFunction = null
}
watcher(function updateTotal() { // 调用watcher() 函数, 更新 data.total
    data.total = data.dinnerPrice + data.tip // = 是 赋值符号
    // = 右侧 是 访问. 会触发getter() 函数
    // function observe(data) {} 内部 代理 的 getter() 函数
    // = 左侧 是 赋值. 会触发setter() 函数
})
// ------------------------------------------------------------------------------

watcher(function render() {
    document.getElementById('app').innerHTML = `
    <table>
        <tr>
            <td>dinner</td>
            <td>${data.dinnerPrice}</td>
            <td>
                <button @click="incrementDinnerPrice">+</button>
                <button @click="decrementDinnerPrice">-</button>
            </td>
        </tr>
        <tr>
            <td>tip</td>
            <td>${data.tip}</td>
            <td>
                <button @click="incrementTip">+</button>
                <button @click="decrementTip">-</button>
            </td>
        </tr>
        <tr>
            <td>total</td>
            <td>${data.total}</td>
            <td>
            </td>
        </tr>
    </table>
    `
})


const methods = {
    incrementDinnerPrice() {
        data.dinnerPrice++
    },
    decrementDinnerPrice() {
        data.dinnerPrice--
    },
    incrementTip() {
        data.tip++
    },
    decrementTip() {
        data.tip--
    }
}
document.getElementById('app')
    .addEventListener('click', function (event) {
        const clickAttr = event.target.attributes['@click']
        const methodName = clickAttr && clickAttr.value
        const method = methods[methodName]
        if (method) {
            method()
        }
    })

```
