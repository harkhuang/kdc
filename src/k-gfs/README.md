## golang 实现简易gfs

#### 设计
1. 
2. 
3. 
4. 
5. 
6. 



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