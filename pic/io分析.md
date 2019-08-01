### IO 路径

RGW网关使用OP线程处理对应的IO请求（OP线程在rgw_main中创建，当前端WEB服务器为CivetWeb时，通过修改配置项rgw_thread_pool_size指定OP线程数目）。  
OP线程内部处理逻辑可分为：HTTP前端、REST API通用处理层、API操作执行层、RADOS接口适配层和librados接口层等几个关键流程。  
OP线程从HTTP前端收到IO请求，首先在REST API通用层，从HTTP语义中解析出S3或者Swift数据并进行一系列检查，检查通过后，根据不同的API操作请求，执行不同打的处理流程。如需从RADOS集群获取数据或者往RADOS集群中写入数据，则通过RGW与Rados接口适配层调用librados接口将请求发送到RADOS集群中获取或写入数据，完成整个IO过程。  

1. HTTP前端
   接收IO请求。
2. REST API通用处理层
   从语义中解析S3或者Swift数据并进行检查
3. API操作执行
   执行处理流程。
4. RGW_RADOS
   调用librados读写数据
5. librados
   读写数据

### 关键过程分析

1. 回调接口：civetweb_calllback
    
    1.1 message家族的结构体
        
        mg_request_info:记录了请求的信息，包括ip,name,port,http_version,uri,user_data等信息。  
        
        mg_connection:记录请求的time信息，buf信息，和上述info等。  

    1.2 
2. 