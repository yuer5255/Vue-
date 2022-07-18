# ETag

为了弥补通过时间戳判断的不⾜，从 **HTTP 1.1** 规范开始新增了⼀个 `ETag` 的头信息。

Etag是服务器为不同资源进⾏哈希运算所⽣成的⼀个字符串，该字符串类似于⽂件指纹，只要⽂件内容编码存在差异，对应的 ETag 标签值就会不同，因此可以使⽤ `ETag` 对⽂件资源进⾏更精准的变化感知。



![](<../.gitbook/assets/image (11).png>)

![](<../.gitbook/assets/image (5) (1).png>)

为了弥补通过时间戳判断的不足，从HTTP 1.1规范开始新增了一个ETag的头信息，即实体标签(EntityTag)。

其内容主要是服务器为不同资源进行哈希运算所生成的一个字符串，该字符串类似于文件指纹，只要文件内容编码存在差异，对应的ETag标签值就会不同，因此可以使用 ETag对文件资源进行更精准的变化感知。下面我们来看一个使用ETag进行协商缓存图片资源的示例，首次请求后的部分响应头

![](<../.gitbook/assets/image (12).png>)

### 演示

```
Router.get('/test4', async (ctx) => {
    const ifNoneMatch = ctx.request.header['if-none-match'];
    const getResource = () => {
        return new Promise((resolve) => {
            fs.readFile(path.join(__dirname, "./fs/a.txt"), (err, data) => {
                if (err) {
                    console.log(err);
                }
                return resolve(etag(data))
            })
        })
    }
    let etagContent = await getResource();
    if (ifNoneMatch === etagContent) { //把具体的日期转换为（根据 GMT）字符串
        ctx.status = 304;
    }
    ctx.set('Etag', etagContent);
    ctx.body = etagContent
})
```



### ETag小结

不像强制缓存中cache-control可以完全替代expires的功能，在协商缓存中， ETag并非last-modified的替代方案而是一种补充方案，因为它依旧存在一些憋端。

* 一方面服务器对于生成文件资源的ETag需要付出额外的计算开销，如果资源的尺寸较大，数量较多且修改比较频繁，那么生成ETag的过程就会影响服务器的性能。
* 另一方面 ETag字段值的生成分为强验证和弱验证，强验证根据资源内容进行生成，能够保证每个字节都相同; 弱验证则根据资源的部分属性值来生成，生成速度快但无法确保每个字节都相同，并且在服务器集群场景下，也会因为不够准确而降低协商缓存有效性验证的成功率，所以恰当的方式是根据具体的资源使用场景选择恰当的缓存校验方式。
