## How Data Is Stored In CEPH Cluster

本文翻译自[How Data Is Stored In CEPH Cluster](https://ceph.com/geen-categorie/how-data-is-stored-in-ceph-cluster/)  

## 数据究竟是如何在Ceph Cluster中存储的？

Now showing a easy to understand ceph data storage diagram.  
    
现在介绍ceph的简单存储图。（翻译的什么鬼……）  


### POOLS

Ceph Cluster有pools，pools是存储对象的逻辑分组。pools由Placement Group组成，创建Pools的过程中，我们要提供pg给pools，部分用于备份。  

+ 创建有128pg的pool
    ```
    pool 'pool_a' created
    ``` 
+ 显示pools
    ```
    ceph osd lspools
    ```
+ 显示pool中的pg数量
    ```
    ceph osd pool get pool-A pg_num
    ```
+ Find out replication level being used by pool( see rep size value for replication )
    ```
    ceph osd dump | grep -i pool-A
    ```
+ 改变备份level
    ```
    # ceph osd pool set pool-A size 3
    set pool 36 size to 3
    #
    # ceph osd dump | grep -i pool-A
    pool 36 'pool-A' rep size 3 min_size 1 crush_ruleset 0 object_hash rjenkins pg_num 128 pgp_num 128 last_change 4054 owner 0
    ```
+ 在pool中上传object
    ```
    rados -p pool-A put object-A  object-A
    ```
+ 检查pools中的objects
    ```
    rados -p pool_a ls
    ```

### Placement Group

Ceph cluster links objects –> PG . These PG containing objects are spread across multiple OSD and improves reliability(提高可靠性)。  

### Object

Object is the smallest unit of data storage in ceph cluster , Each & Everything is stored in the form of objects , thats why ceph cluster is also known as Object Storage Cluster. Objects are mapped to PG , and these Objects / their copies always spreaded on different OSD. This is how ceph is designed.  

object 是最小的单元，这也就是为什么称之为对象存储集群的原因。objects映射到了placement group，objects和其复制本，被存储在不同的OSD上。  

+ object属于哪个PG？
    ```
    ceph osd map pool-A object-A
    /*
    osdmap e4055(osd映射的版本id是e4055) 
    pool 'pool-A' (poolname:pool-A,pool_id:36) 
    object 'object-A'(obj_name) -> 
    pg 36.b301e3e8 (pg id:36.68) 
    -> up [122,63,62] acting [122,63,62]
    (复制等级为3，因此有三个备份，分别在OSD：122,63,62)
    */
    ```

+ 查看具体的路径
    ```
    /*去具体的查看OSD挂载的路径，并且去具体的路径下查看*/
     df -h /var/lib/ceph/osd/ceph-122

     cd /var/lib/ceph/osd/ceph-122

    ls -la | grep -i 36.68

    ls -l
    /*
    total 10240
    -rw-r--r-- 1 root root 10485760 Jan 24 16:45 object-A__head_B301E3E8__24
    */
    ```

### Moral of the Story（怎么翻译？）

+ ceph存储集群可以有多个pools  
+ 每一个pools都应有多个PG， More the PG , better your cluster performance , more reliable your setup would be.  
+ 一个PG有多个objects  
+ 一个PG跨存在多个OSD上。例如：objects扩展在OSD上，第一个mapped（映射）到PG的OSD应该是基本的OSD，并且同意PG对应的其他OSD，是第二OSD（翻译的鬼一样，大致意思就是，Object不会存储在一个OSD上，确保数据的安全性）  
+ 一个object可以只映射到一个PG上（如果没有备份的话）
+ PG和OSD的对应关系是n--->1

### 一个Pool需要多少placements？

```
             (OSDs * 100)
Total PGs = -----------------
            Replicas（副本数）
```

查看OSD的状态
```
ceph osd stat
//显示结果：osdmap e4055: 154 osds: 154 up, 154 in
```
( 154 * 100 ) / 3 = 5133.33，然后最接近2的次幂的数字就行了。8192PGs  

