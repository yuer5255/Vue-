# Last-Modifed

![](<../.gitbook/assets/image (14) (1).png>)

在浏览器第一次请求时，服务器返回资源和资源标识符Last-Modified(在响应头中)

![](<../.gitbook/assets/image (17) (1).png>)

在后续请求中，浏览器就会在请求头带上资源标识If- Modified-Since发起请求，If-Modified-Since的值就是上一次请求时返回的Last-Modified的值，这时在服务器就会去对比IfModified-Since和Last-Modified的值判断是否是最新资源。



### 演示

```
Router.get('/test3', async (ctx) => {
    const ifModifiedSince = ctx.request.header['if-modified-since'];
    const getResource = () => {
        return new Promise((resolve) => {
            fs.stat(path.join(__dirname, "./fs/a.txt"), (err, stats) => {
                if (err) {
                    console.log(err);
                }
                return resolve(stats)
            })
        })
    }
    let resource = await getResource();
    if (ifModifiedSince === resource.mtime.toUTCString()) { //把具体的日期转换为（根据 GMT）字符串
        ctx.status = 304;
    }
    ctx.set('Last-Modified', resource.mtime.toUTCString());
    ctx.body = resource
})
```



### Last-Modified 小结

通过last-modified所实现的协商缓存能够满足大部分的使用场景，但也存在两个比较明显的缺陷:

* 首先它只是根据资源最后的修改时间戳进行判断的，虽然请求的文件资源进行了编辑，但内容并没有发生任何变化，时间戳也会更新，从而导致协商缓存时关于有效性的判断验证为失效，需要重新进行完整的资源请求。这无疑会造成网络带宽资源的浪费，以及延长用户获取到目标资源的时间。&#x20;
* 其次**标识文件资源修改的时间戳单位是秒**，如果文件修改的速度非常快，假设在几百毫秒内完成，那么上述通过时间戳的方式来验证缓存的有效性，是无法识别出该次文件资源的更新的



其实造成上述两种缺陷的原因相同，就是服务器无法仅依据资源修改的时间戳来识别出真正的更新，进而导致重新发起了请求，该重新请求却使用了缓存的Bug场景。

