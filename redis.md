#  redis



- String

  - 什么是sds
  - 什么是二进制安全

- list

  - 3.2之前 linkedlist  ziplist(数据量小的时候)

  - 3.2以后quickList 双向链表  linkedList+ziplist

  - list使用场景

    - 分布式队列

      

![1536759712086](C:\Users\ADMINI~1\AppData\Local\Temp\1536759712086.png)

- hash类型
  - 参见hashMap实现	
- set
  - intset hashtable(key,value(null))
  - 集合
- sortset（有序集合）
  - 数据结构
    - ziplist 或者skiplist+hashtable