## golang 实现简易gfs

#### 设计
首先明确设计的目的 
```使得随机并发读取不再收到单个计算机io限制```

1. 在有了mapreduce(分布式计算框架-并行有序) 和 bigtbale(分布式读写一致性kv)基础上
2. 为了简化gfs对io接口,只做read random chunk操作 ,只做wirete append chunk操作 避免了chunk之间关系过于复杂
3. 模块master仍然是派发接口工作的主要进程,为了简化设计,仍然他只有一个,而且不考虑高可用.这样简化了我们大部分的设计工作
4. 模块master用树结构存储了分布式文件系统的目录和节点信息,和其中一个主要的chunk-server获取通信,获取到所有的模块信息,这样我们就可以读取整个分布式文件了.
5. 模块chunk-server分主次,主模块调度次要模块,找到索引分别和client建立链接传输数据,次要模块只和master保持心跳不做数据传输




#### 思考

如何做到并发写操作??   虽然是追加  我们先在master中生成有序的index-  那么我们就可以做分布式的写操作了,当然执行这些之前一定是做好文件分块操作了... 但是如果每次写的数据很小4M写了256次 就不太适合分布式文件写操作了.  但是我们能够缩小chunk的大小我们仍然可以做一个分布式的图片文件处理.....



#### 提问 提问:
问题: 小文件读写瓶颈不再io 而是在于cpu的计算速度?  如果用nginx的反向代理俩做这件事情,会又什么不同之处呢?? 是否可以做横向比较?

####   master 内存中数据的结构
 + chunk namespace
 + file namespace
 + file <---> chunk 映射map
 + chunk副本存放位置信息

file1->(chunk1   chunk3   chunk5)
读文件 ->master-> sync (获取分布式读锁)chunk1-client -chunk2-client chunk3-client
master-getret->merge
写文件 ->master->选择三个可靠节点做写入操作中->记录文件信息在master-index
删除文件->获取全局的写锁->删除对应节点chunk->删除master的index



######  一些问题
1. 节点失效?
2. 节点恢复?
3. 节点任务派发?
4. 粗粒度锁头解释和实现原理?



