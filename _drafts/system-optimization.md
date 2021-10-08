## 高性能架构

### Optimize web frontend

#### 浏览器访问优化

1. 减少HTTP请求：合并CSS、合并JavaSript、合并图片。
2. 使用浏览器缓存：设置HTTP头中的 Cache-Control 和 Expires。更新时可修改JS的文件名，并更新HTML中的引用。多个文件更新时，可考虑批量更新，避免用户浏览器大量缓存失效，集中更新造成服务器负载和网络堵塞等。
3. 启用压缩：可对HTML、CSS、JavaSript文件启用gzip压缩。需衡量服务器CPU资源和带宽占用情况。
4. CSS放页面最上面，JavaSript放页面最下面：浏览器在下载完CSS后才对页面进行渲染，CSS在上面可以让浏览器尽快下载CSS。浏览器在加载完JavaSript后立即执行，可能会阻塞整个页面，造成显示缓慢。但是如果页面解析时就要用到的JavaSript则不能放下面。
5. 减少cookie传输。

#### CDN (Content Distribute Network)

CDN 一般用来缓存静态资源，如图片、文件、CSS、静态网页等。

#### Reverse Proxy

1. 安全性。在web服务器和外网间建立了屏障。
2. 配置缓存可加速web请求。通过内部机制通知反向代理缓存失效。

### Optimize application service

#### 缓存的基本原理

数据以Key、Value的形式存储的内存的Hash table中。对Key先做Hash code，再找索引。读写的时间复杂度是O(1)。适用于读写比很高，很少变化的数据。

#### 合理使用缓存

滥用缓存可能让缓存成为系统的风险点。下面的情况需注意：

1. 频繁修改的数据。数据的读写比在2:1以上，缓存才有意义。
2. 没有热点的访问。由于内存限制，不能缓存所有的数据，如果大多数数据还没有访问到就被清理出缓存了，则缓存没有意义。
3. 数据不一致和脏读。需考虑设置缓存失效时间，缓存更新策略。
4. 缓存可用性。发生cache avalanche或者redis挂了。cache数据的expired时间可以加入随机数据，cache down了以后需要对服务做降级、限流，master slave部署cache。
5. 缓存预热 cache warming。可在缓存上线前预先加载好数据，避免大批的对cache的访问miss了，然后压力都转移到了DB中。
6. 缓存穿透 cache penetration。错误的业务或者黑客攻击导致请求的cache无效，压力都被转移到DB上。可以缓存失效的key，可以使用bloom filter。

#### 代码优化

1. 多线程：CPU计算型任务，线程数最好不超过CPU core的数量。注意锁的使用。
2. 资源复用：尽量减少系统资源的创建和销毁，比如数据库连接、网络连接、线程、复杂对象等。
3. 数据结构、算法。
4. JVM需注意垃圾回收、参数调优。

### Optimize storage

