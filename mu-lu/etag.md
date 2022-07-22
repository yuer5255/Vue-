# ETag

为了弥补通过时间戳判断的不⾜，从 **HTTP 1.1** 规范开始新增了⼀个 `ETag` 的头信息。

`Etag`是服务器为不同资源进⾏哈希运算所⽣成的⼀个字符串，该字符串类似于⽂件指纹，只要⽂件内容编码存在差异，对应的`ETag` 标签值就会不同，因此可以使⽤ `ETag` 对⽂件资源进⾏更精准的变化感知。



![](<../.gitbook/assets/image (11).png>)

![](<../.gitbook/assets/image (5) (1).png>)

![](<../.gitbook/assets/image (12).png>)

### 演示

```javascript
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

不像强制缓存中`cache-control`可以完全替代`expires`的功能，在协商缓存中， `ETag`并非`last-modified`的替代方案而是一种补充方案，因为它依旧存在一些弊端。

* 一方面服务器对于生成文件资源的ETag需要付出额外的计算开销，如果资源的较大，数量较多且修改比较频繁，那么生成ETag的过程就会影响服务器的性能。
* 另一方面 ETag字段值的生成分为强验证和弱验证，强验证根据资源内容进行生成，能够保证每个字节都相同; 弱验证则根据资源的部分属性值来生成，生成速度快但无法确保每个字节都相同，并且在服务器集群场景下，也会因为不够准确而降低协商缓存有效性验证的成功率，所以恰当的方式是根据具体的资源使用场景选择恰当的缓存校验方式。
