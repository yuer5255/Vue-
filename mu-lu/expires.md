# Expires

{% hint style="info" %}
expires 是在HTTP 1.0 协议中声明的用来控制缓存失效日期时间戳的字段，它由服务器端指定后通过**响应头**告知浏览器，浏览器在接收到带有该字段的响应体后进行缓存。
{% endhint %}

若之后浏览器再次发起相同的资源请求，便会对比  `Expires` 与客户端当前的时间戳。

如果当前请求的本地时间戳小于 `Expires`的值，则说明浏览器缓存的响应还未过期，可以直接使用缓存资源，而无须向服务器端再次发起请求。

而当本地时间戳大于`Expires` 值，代表缓存过期时，才允许重新向服务器发起请求。

### 演示

```javascript
Router.get('/test1', async (ctx) => {
    const getResource = () => {
        return new Promise((resolve) => {
            fs.readFile(path.join(__dirname, "./fs/a.txt"), (err, data) => {
                if (err) {
                    return;
                }
                return resolve(data.toString())
            })
        })
    }
    ctx.set('Expires', new Date('2022-7-18 9:50:00').toUTCString())  //设置强缓存，过期时间需要自己设置
    ctx.body = await getResource();
})
```



### 小结

从上述强制缓存是否过期的判断机制中不难看出，这个方式存在一个很大的漏洞.

即对**本地时间戳过分依赖，如果客户端本地的时间与服务器端的时间不同步，或者对客户端时间进行主动修改，那么对于缓存过期的判断可能就无法和预期相符。**

解决这一个问题，我们再了解一下另一个与强缓存相关的字段`Cache-Control`
