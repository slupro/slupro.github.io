# Designing Data Intensive Applications (DDIA)

In DDIA, it focus on 3 concerns:

> 1. Reliability: Continuing to work correctly, even when things go wrong.
> 2. Scalability: Scale up, or scale out.
> 3. Maintainability: Operability, Simplicity, Extensibility.


The architecture of systems that operate at large scale is usually highly specific to the
application—there is no such thing as a generic, one-size-fits-all scalable architecture
(informally known as magic scaling sauce). The problem may be *the volume of reads*,
*the volume of writes*, *the volume of data to store*, *the complexity of the data*, *the
response time requirements*, *the access patterns*, or (usually) some mixture of all of
these plus many more issues.



> Fault vs Failure

A fault is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user.

Document model: Document databases are sometimes called schemaless, but that’s misleading, as the code that reads the data usually assumes some kind of structure—i.e., there is an
implicit schema, but it is not enforced by the database. A more accurate term is
schema-on-read (the structure of the data is implicit, and only interpreted when the
data is read), in contrast with schema-on-write (the traditional approach of relational databases, where the schema is explicit and the database ensures all written data con‐
forms to it).

## Storage

### Hash map做数据索引

以log appending的形式记录数据，使用Hash->offset的方式来在内存中创建index。数据都是在前一个记录后做追加。到一定时候再创建新的数据文件，后台有进程定期merge之前的数据文件成新的数据库文件。merge时对同样的key，只保留最后的数据，相当于update只记录最终结果。

![](/assets/img/DDIA-notes/2021-11-21-23-07-55.png)

#### 实现时需要考虑

1. File format，可以是KLV里面的LV。
2. Deleting records. 由于是appending形式记录，所以删除不能直接删，而是appending一个特殊的删除标记。merge的时候，有删除标记的记录会被丢弃。
3. Crash recovery. index的hash map是在内存中，如果数据库重启，需要读取所有数据库文件重建内存的index，所以可以将hash map的snapshot也保存在磁盘上。
4. Partially written records. 如果写入一半数据时崩溃，需要在数据库文件中包括checksum，以检测到数据库文件不完整，以丢弃不完整的数据。
5. Concurrency control. 追加是严格的序列化操作，所以可以用一个线程负责写入，多个线程读取。

#### 为什么用appending而不是update

1. appending可以利用sequential write，而不是update的random write。
2. 对并发和恢复的处理更简单。
3. update更容易让文件碎片化。Merging old segments avoids the problem of data files getting fragmented over time.

#### Limitations

1. hash map在内存中，如果key数量很大则内存不足以处理。但在磁盘上维护hash map，效率不高，需要很多random access I/O，并且需要处理hash collisions。
2. Range queries are not efficient.

### SSTables and LSM-Trees

Sorted String Table (SSTables) 和以Hash map做索引不同，SSTables要求对key进行排序，并有以下优点：

1. Merge的实现可以简单高效。
2. 在文件中查找指定的key时，不需要把所有key的index都加载到内存中，只需要一个稀疏索引在内存中。You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly.
3. 由于读请求经常需要扫描连续的一段范围，所以可以将相邻的块压缩后再写入磁盘，从而节省磁盘空间和I/O。

![](/assets/img/DDIA-notes/2021-11-23-21-14-27.png)

#### 构建和维护SStables

被用于LevelDB、RocksDB，类似的算法也被用于Cassandra和HBase。

1. 写入数据时，add it to an in-memory balanced tree data structure (eg, a red-black tree, or AVL tree).
2. 当memtable超过一个threshold时，落盘。树已经维护了按key排序的key-value对，所以顺序落盘即可。有新数据来时，创建新的memtable。
3. 处理读请求时，先从memtable里面找，再按磁盘文件的新旧顺序一次查找。
4. 后台进程周期性地merge。

> 为了确保memtable中的数据丢失，可以在磁盘上保存一个单独的日志，每个写入都追加。当数据库崩溃时，可用来恢复memtable。

#### LSM-Tree (Log-Structured Merge-Tree)

基于Merge和Compact sorted files的存储引擎都被成为 LSM storage engine。

> 性能优化：LSM engine在搜索一个不存在的key时，会很慢，因为会一直查找到最老的文件。优化方案是使用bloom filter，排除不存在的key。

#### B-Tree

The log-structured indexes we saw earlier break the database down into variable-size segments, typically several megabytes or more in size, and always write a segment sequentially. By contrast, B-trees break the database down into fixed-size blocks or pages, traditionally 4 KB in size (sometimes bigger), and read or write one page at a time.

![](/assets/img/DDIA-notes/2021-11-23-22-53-15.png)

添加新key时，若无足够的可用空间，则将旧页分裂为两个页。该算法保证了树的balanced。通常有三到四层B-tree，一个4层的b-tree(4KB page)可以存储256TB。

![](/assets/img/DDIA-notes/2021-11-23-22-57-53.png)

