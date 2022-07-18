# Cache-Control

{% hint style="info" %}
为了解决 **expires**判断的局限性，从HTTP 1.1协议开始新增了**cache-control**字段来对expires的功能进行扩展和完善。
{% endhint %}

![](<../.gitbook/assets/image (9).png>)

### 演示

```
Router.get('/test2', async (ctx) => {
    const getResource = () => {
        return new Promise((resolve) => {
            fs.readFile(path.join(__dirname, "./fs/a.txt"), (err, data) => {
                console.info('err', err);
                if (err) {
                    return;
                }
                return resolve(data.toString())
            })
        })
    }
    console.info('aaa', await getResource());
    ctx.set('Cache-Control', 'max-age=10')  //设置强缓存，过期时间为10秒
    ctx.body = await getResource();
})
```



从上述代码中可见cache-control设置了 **maxage=10**的属性值来控制响应资源的有效期，它是一个以秒为单位的时间长度，**表示该资源在被请求到后的10秒内有效**，如此便可避免服务器端和客户端时间戳不同步而造成的问题。

from memory cache 代表使用内存中的缓存，from disk cache 则代表使用的是硬盘中的的缓存，浏览器读取缓存的顺序为memory->disk。



除此之外， cache-control 还可配置一些其他属性值来更准确地控制缓存，下面来具体介绍。

cache-control的几个属性

![](<../.gitbook/assets/image (8) (1).png>)

### no-cache和no-store

设置no-cache并非像字面上的意思不使用缓存，其表示为强制进行协商缓存(后面会说)，即对于每次发起的请求都不会再去判断强制缓存是否过期，而是直接与服务器协商来验证缓存的有效性，若缓存未过期，则会使用本地缓存。

设置no-store则表示禁止使用任何缓存策略，客户端的每次请求都需要服务器端给予全新的响应。

no-cache和no-store是两个互斥的属性值，不能同时设置。



### private和public

private和public也是cache-control的一组互斥属性值，它们用以明确响应资源是否可被代理服务器进行缓存。

若资源响应头中的cache-control字段设置了 public属性值，则表示响应资源既可以被浏览器缓存，又可以被代理服务器缓存。 private 则限制了响应资源只能被浏览器缓存，若未显式指定则默认值为private



### max-age和s-maxage

max-age属性值会比 s-maxage更常用，它表示服务器端告知客户端浏览器响应资源的过期时长。在一般项目的使用场景中基本够用，对于大型架构的项目通常会涉及使用各种代理服务器的情况，这就需要考虑缓存在代理服务器上的有效性问题。这便是s- maxage存在的意义，它表示缓存在代理服务器中的过期时长，且仅当设置了public属性值时才有效。



### 总结

* 优先级： **cache-control > expires**
* cache-control 能作为 expires 的完全替代方案，并且拥有其所不具备的一些缓存控制特性，在项目实践中使用它就足够了。
* 目前 expires还存在的唯一理由是考虑可用性方面的向下兼容。
