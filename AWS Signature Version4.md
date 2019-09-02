## AWS Signature Version 4

s3协议下的所有接口，都是验证过或者匿名的操作。本文介绍了AWS签名V4算法下的验证请求。  
Every interaction with Amazon S3 is either authenticated or anonymous. This section explains request  
authentication with the AWS Signature Version 4 algorithm.  

V4算法适用于以下情况：

1. Verification of the identity of the requester

验证请求需要用户提供一个签名，签名通过秘钥（公钥access key ID，私钥secret access key）计算。如果是临时通行证，签名的计算还需要安全口令token。  

2. In-transit data peotection

为了避免传输过程中的请求篡改，你需要一些请求中的元素计算签名，s3使用童谣的元素计算签名。

3. protect against reuse of the signed portions of the request

为了避免请求的signed portions被重复使用，因此时间戳限定在15分钟内。  
在15分钟内，没有授权的party，在有签名请求的权限下，可以修改未签名部分的请求，同时不影响请求的合法性。  
正因为如此，我们建议同时加密请求的header和body，使用HTTPS的请求，使用s3:x-amz-content-sha256 condition key。

#### 说明

Amazon S3 在所有的AWS领域内，支持V4签名算法（AWS服务所需要的请求验证协议）。2014.1.30之前创建的AWS regions将会继续支持V2算法，但是之后创建的AWS Regions只支持V4协议。  
#### 验证方法

验证信息的组织有以下两种方法：
1. HTTP Authorization header

使用HTTP验证头是最常用的方法。所有Amazon Ss3 REST操作需要这个头信息，除了**基于浏览器，使用POST请求的上传**

2. Query string parameters

在URL中使用query字符串，如果这样的话，使用query parameters提供请求信息，其中包括验证信息，因为清明签名是URL的一部分，这种URL也常用作预签名的的URL。  
将上述URLs嵌入到可点击的链接中，在HTML中，有效期是7天。

#### 介绍--签名请求

请求中的验证信息中必须有签名，为了计算签名，首先需要规范化选择请求元素，组成一个字符串，这一步成为string-to-sign。  
然后用signing key计算string-to sign 的 HMAC（hash-based message authentication code）.  
在V4协议中，不必使用私钥去sign请求；相反，你首先需要用私钥创造一个signing-key，这个key作用于特殊region和服务，永不到期。  

如图：  
![avatar](./pic/authenticate/sign-request.png)

string-to-sign依赖于请求类型。比如说，HTTP的授权头或者验证请求参数，可以使用不同的组合请求元素，重建string-to-sign。  
HTTP POST请求，请求中的POST policy就是自己sign的。


### Authenticating Requests: Using the Authorization Header(AWS Signature Version 4)

提供验证信息最常用的办法就是使用HTTP 授权Header。除了**POST请求和使用请求参数签名的请求**，所有的S3对象或桶操作使用授权请求头，提供验证信息。  

授权Header例子如下（中间添加回车，便于阅读）：

```
Authorization: AWS4-HMAC-SHA256
Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/s3/aws4_request,
SignedHeaders=host;range;x-amz-date,
Signature=fe5f80f77d5fa3beca038a248ff027d0445342fe2855ddc963176630326f1024
```

参数解释：

| component        | description |
| :--------------- | :---------- |
| AWS4-HMAC-SHA256 | 计算签名的算法，验证必备。<br>指定AWS4和HMAC-SHA256 |
| Credential       | 公钥ID和周边信息，包含date，region，计算签名的服务。如下：<br> your-access-key-id/date/aws-region/aws-service/aws4_request <br> date:YYMMDD格式 <br> aws-service 如果是发送给S3，之就是s3 |
| SignedHeaders    | 分号相隔开的一个list，用以计算签名，包含了header-name，必须是小写：<br> host;range;x-amz-date |
| Signature        | 256位的签名，由64位16进制的字符组成，例如：<br> fe5f80f77d5fa3beca038a248ff027d0445342fe2855ddc963176630326f1024 <br>签名计算取决于你选择传输payload的option |


当你提起一个请求，必须告诉S3，你是用的那种选项计算签名算法，使用x-amz-content-sha256头信息加备注的形式，比如：

**chunked upload options**: header-value设置为：STREAMING-AWS4-HMAC-SHA256-PAYLOAD  
**a single chunk upload options**: header-value添加signed payload 选项，或者字面添加UNSIGNED-PAYLOAD  


#### Transfer payload in a single chunk 

有以下两种签名计算选项：  

1. Signed payload option
2. Unsigned payload option


#### Transfer payload in multiple chunks (chunked upload)

In this case you transfer payload in chunks.You can transfer a payload in chunks regardless of the payload size.  


### Signature Calculations for the Authorization Header:Transferring Payload in a Single Chunk (AWS Signature Version 4)

使用授权的header去验证请求时，header-value会包含一个签名，签名的计算方法取决于你提供的payload，这一部分介绍“Transfer payload in a single chunk”的签名计算方法。  

#### important

分为两种情况：

1. Signed payload option

2. Unsigned payload option

When you send your request to S3, the x-amz-content-sha256 header value informs S3  
whether the payload is signed or not. Amazon S3 can then create signature accordingly for verification  

#### Calculating a Signature

1. 首先需要string-to-sign
2. 使用signing-key计算string-to-sign的HMAC-SHA256 hash值

当S3接收到验证请求，它会计算签名并对比请求中提供的签名。因此必须使用同样的签名计算方法。  
规范化就是把请求的格式进行统一的过程。  

![avatar](./pic/authenticate/calculate-signature.png)

##### Task 1: Create a Canonical Request

```
<HTTPMethod>
<CanonicalURI>
<CanonicalQueryString>
<CanonicalHeaders>
<SignedHeaders>
<HashedPayload>
```

解释：  
1. HTTPMethod

is one of the HTTP methods, for example GET, PUT, HEAD, and DELETE.  

2. CanonicalURI

URI-encode的组织形式

```
http://s3.amazonaws.com/examplebucket/myphoto.jpg  
```

3. CanonicalQueryString

就是URI后面跟的参数

```
http://s3.amazonaws.com/examplebucket?prefix=somePrefix&marker=someMarker&max-keys=20  
```

规范化的query-string 如下：  

```
UriEncode("marker")+"="+UriEncode("someMarker")+"&"+
UriEncode("max-keys")+"="+UriEncode("20") + "&" +
UriEncode("prefix")+"="+UriEncode("somePrefix")
```

如果是subresource,格式如下：

```
http://s3.amazonaws.com/examplebucket?acl
UriEncode("acl") + "=" + ""
```

4. CanonicalHeaders

一系列的请求头和对应的value。  

```
Lowercase(<HeaderName1>)+":"+Trim(<value>)+"\n"
Lowercase(<HeaderName2>)+":"+Trim(<value>)+"\n"
...
Lowercase(<HeaderNameN>)+":"+Trim(<value>)+"\n"
```

必须包含**HTTP host header**；  
如果提供了content-type，则必须添加到CanonicalHeaders的list中；  
x-amz-* headers 也必须加进去。例如，如果是**临时用户**，必须加上x-amz-security-token；  
x-amz-content-sha256是所有AWS Signature Version 4都需要的，


5. SignedHeaders

用";"分割的小写字符串。  

```
host;x-amz-content-sha256;x-amz-date
```

6. HashedPayload

the SHA256 hash的16进制表达式,如果没有payload，用""代替。  

```
Hex(SHA256Hash(<payload>)
Hex(SHA256Hash(<"">)
/*return value*/
//e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

##### Task 2: Create a String to Sign

连接一系列字符串，构建一个新的字符串的过程

```
"AWS4-HMAC-SHA256" + "\n" +
timeStampISO8601Format + "\n" +
<Scope> + "\n" +
Hex(SHA256Hash(<CanonicalRequest>))
```

时间格式：  

```
date.Format(<YYYYMMDD>) + "/" + <region> + "/" + <service> + "/aws4_request"
/*the resulting signature to the us-east-1 region and Amazon S3*/
//eg:   20130606/us-east-1/s3/aws4_request
```

#### Task 3: Calculate Signature

The final signature is the HMAC-SHA256 hash of the **string to sign**, using the **signing key** as the key.

```
HMAC-SHA256(SigningKey, StringToSign)
```

#### Example

+ Example access keys:  
| Parameter | Value |
| :-------- | :---- |
| AWSAccessKeyId | AKIAIOSFODNN7EXAMPLE |
| AWSSecretAccessKey | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY |

+ Request timestamp of 20130524T000000Z (Fri, 24 May 2013 00:00:00 GMT).  

+ Bucket name : examplebucket.

+ 时区：US East (N. Virginia)，使用us-east-1作为地区识别。  

+ sign a virtual hosted-style request : https://examplebucket.s3.amazonaws.com/photos/photo1.jpg

##### get object

The following example gets the first 10 bytes of an object (test.txt) from examplebucket.

```
GET /test.txt HTTP/1.1
Host: examplebucket.s3.amazonaws.com
x-amz-date:20130524T000000Z
Authorization: SignatureToBeCalculated
Range: bytes=0-9
x-amz-content-sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
x-amz-date: 20130524T000000Z
```

由于GET请求不提供body信息，x-amz-content-sha256的value是取空串的sha-256。  

1. StringToSign
   1. CanonicalRequest
   
    ```
    GET
    /test.txt

    host:examplebucket.s3.amazonaws.com
    range:bytes=0-9
    x-amz-content-
    sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    x-amz-date:20130524T000000Z

    host;range;x-amz-content-sha256;x-amz-date
    e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    ```

    第三行是空，因为没有请求参数。

   2. StringToSign

    ```
    AWS4-HMAC-SHA256
    20130524T000000Z
    20130524/us-east-1/s3/aws4_request
    7344ae5b7ee6c3e7e6b0fe0640412a37625d1fbfff95c48bbb2dc43964946972
    ```

2. SigningKey

```
signing key = HMAC-SHA256(HMAC-SHA256(HMAC-SHA256(HMAC-SHA256("AWS4" +
"<YourSecretAccessKey>","20130524"),"us-east-1"),"s3"),"aws4_request")
```

3. Signature

```
f0e8bdb87c964420e857bd35b5d6ed310bd44f0170aba48dd91039c6036bdb41
```


4. Authorization header

Authorization header：

```
AWS4-HMAC-SHA256 Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/
s3/aws4_request,SignedHeaders=host;range;x-amz-content-sha256;x-amz-
date,Signature=f0e8bdb87c964420e857bd35b5d6ed310bd44f0170aba48dd91039c6036bdb41
```

##### put object

This example PUT request creates an object (test$file.text) in examplebucket.  

x-amz-storage-class设置为REDUCED_REDUNDANCY  
file.txt的内容是：“Welcome to Amazon S3.”。x-amz-content-sha256的值就是基于此进行计算。  

```
PUT test$file.text HTTP/1.1
Host: examplebucket.s3.amazonaws.com
Date: Fri, 24 May 2013 00:00:00 GMT
Authorization: SignatureToBeCalculated
x-amz-date: 20130524T000000Z
x-amz-storage-class: REDUCED_REDUNDANCY
x-amz-content-sha256: 44ce7dd67c959e0d3524ffac1771dfbba87d2b6b4b4e99e42034a8b803f8b072
<Payload>
```

1. StringToSign
   1. CanonicalRequest
   
    ```
    PUT
    /test%24file.text

    date:Fri, 24 May 2013 00:00:00 GMT
    host:examplebucket.s3.amazonaws.com
    x-amz-content-sha256:44ce7dd67c959e0d3524ffac1771dfbba87d2b6b4b4e99e42034a8b803f8b072
    x-amz-date:20130524T000000Z
    x-amz-storage-class:REDUCED_REDUNDANCY
    date;host;x-amz-content-sha256;x-amz-date;x-amz-storage-class
    44ce7dd67c959e0d3524ffac1771dfbba87d2b6b4b4e99e42034a8b803f8b072
    ```

    > 第三行是空，因为没有请求参数。最后一行是body的hash值，应该和x-amz-content-sha256 header value一致。

   2. StringToSign

    ```
    AWS4-HMAC-SHA256
    20130524T000000Z
    20130524/us-east-1/s3/aws4_request
    9e0e90d9c76de8fa5b200d8c849cd5b8dc7a3be3951ddb7f6a76b4158342019d
    ```
    
    **计算方法Note:**

    ```
    "AWS4-HMAC-SHA256" + "\n" +
    timeStampISO8601Format + "\n" +
    <Scope> + "\n" +
    Hex(SHA256Hash(<CanonicalRequest>))
    ```
    
2. SigningKey

```
signing key = HMAC-SHA256(HMAC-SHA256(HMAC-SHA256(HMAC-SHA256("AWS4" +
"<YourSecretAccessKey>","20130524"),"us-east-1"),"s3"),"aws4_request")
```

3. Signature

```
98ad721746da40c64f1a55b78f14c238d841ea1380cd77a1b5971af0ece108bd
```

**计算方法Note:**

```
DateKey = HMAC-SHA256("AWS4"+"<SecretAccessKey>", "<YYYYMMDD>")
DateRegionKey = HMAC-SHA256(<DateKey>, "<aws-region>")
DateRegionServiceKey = HMAC-SHA256(<DateRegionKey>, "<aws-service>")
SigningKey = HMAC-SHA256(<DateRegionServiceKey>, "aws4_request")
```

**计算方法Note:**

> Signature = HMAC-SHA256(SigningKey, StringToSign)

4. Authorization header

```
AWS4-HMAC-SHA256 
Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/s3/aws4_request,
SignedHeaders=date;host;x-amz-content-sha256;x-amz-date;x-amz-storage-class,
Signature=98ad721746da40c64f1a55b78f14c238d841ea1380cd77a1b5971af0ece108bd
```

### Signature Calculations for the Authorization Header:Transferring Payload in Multiple Chunks (Chunked Upload) (AWS Signature Version 4)

Each chunk signature calculation includes the signature of the previous chunk.  
每一块的计算都会包含前一个块的签名。因此需要一个**种子签名**。第一个块的签名计算用**种子签名**。这种链状结构保证了chunk的正确顺序。  

1. Decide the payload chunk size. You need this when you write the code.  
2. Create the seed signature for inclusion in the first chunk.
3. Create the first chunk and stream it.
4.  For each subsequent chunk
    > calculate the chunk signature that includes the previous signature in the string you sign, construct the chunk, and send it.
5. Send the final additional chunk
    > which is the same as the other chunks in the construction, but it has zero data bytes. For more information

#### Calculating the Seed Signature

和单块操作的计算方法一致，除了在CanonicalRequest过程中有所不同：  

1. x-amz-content-sha256

设置为STREAMING-AWS4-HMAC-SHA256-PAYLOAD  

2. Content-Encoding

Content-Encoding : aws-chunked,gzip  

3. x-amz-decoded-content-length

记录数据的长度  

4. Content-Length

设置实际传输的HTTP body的长度  

![avatar](./pic/authenticate/seed-signature.png)

#### Defining the Chunk Body

All chunks include some metadata. Each chunk must conform to the following structure:  

```
string(IntHexBase(chunk-size)) + ";chunk-signature=" + signature + \r\n + chunk-data + \r\n
```

IntHexBase() is a function that you write to convert an integer chunk-size to hexadecimal.   
For example, if chunk-size is 65536, hexadecimal string is "10000".

chunk-size is the size, in bytes, of the chunk-data, without metadata. For example, if you are  
uploading a 65 KB object and using a chunk size of 64 KB, you upload the data in three chunks: the  
first would be 64 KB, the second 1 KB, and the final chunk with 0 bytes.  

#### Example: PUT Object

You are uploading a 65 KB text file, and the file content is a one-character string made up of the letter 'a'.  
You are requesting REDUCED_REDUNDANCY as the storage class by adding the x-amz-storage-class request header.  
The resulting object has the key name chunkObject.txt.  
文件大小65k，每次上传64k，分为三次，64k~1k~0k

```
PUT /examplebucket/chunkObject.txt HTTP/1.1
Host: s3.amazonaws.com
x-amz-date: 20130524T000000Z
x-amz-storage-class: REDUCED_REDUNDANCY
Authorization: SignatureToBeCalculated
x-amz-content-sha256: STREAMING-AWS4-HMAC-SHA256-PAYLOAD
Content-Encoding: aws-chunked
x-amz-decoded-content-length: 66560
Content-Length: 66824
<Payload>
```

1. Seed signature — Create String to Sign
   1. CanonicalRequest
   
    ```
    PUT
    /examplebucket/chunkObject.txt
    content-encoding:aws-chunked
    content-length:66824
    host:s3.amazonaws.com
    x-amz-content-sha256:STREAMING-AWS4-HMAC-SHA256-PAYLOAD
    x-amz-date:20130524T000000Z
    x-amz-decoded-content-length:66560
    x-amz-storage-class:REDUCED_REDUNDANCY
    content-encoding;content-length;host;x-amz-content-sha256;x-amz-date;x-amz-decoded-content-length;x-amz-storage-class
    STREAMING-AWS4-HMAC-SHA256-PAYLOAD
    ```

    > 第三行是空，因为没有请求参数。最后一行是Payload的hash值，应该和x-amz-content-sha256 header value一致。

   2. StringToSign

    ```
    AWS4-HMAC-SHA256
    20130524T000000Z
    20130524/us-east-1/s3/aws4_request
    cee3fed04b70f867d036f722359b0b1f2f0e5dc0efadbc082b76c4c60e316455
    ```
    
    **计算方法Note:**

    ```
    "AWS4-HMAC-SHA256" + "\n" +
    timeStampISO8601Format + "\n" +
    <Scope> + "\n" +
    Hex(SHA256Hash(<CanonicalRequest>))
    ```
    
2. SigningKey

```
signing key = HMAC-SHA256(HMAC-SHA256(HMAC-SHA256(HMAC-SHA256("AWS4" +
"<YourSecretAccessKey>","20130524"),"us-east-1"),"s3"),"aws4_request")
```

3. Signature

```
4f232c4386841ef735655705268965c44a0e4690baa4adea153f7db9fa80a0a9
```

**计算方法Note:**

```
DateKey = HMAC-SHA256("AWS4"+"<SecretAccessKey>", "<YYYYMMDD>")
DateRegionKey = HMAC-SHA256(<DateKey>, "<aws-region>")
DateRegionServiceKey = HMAC-SHA256(<DateRegionKey>, "<aws-service>")
SigningKey = HMAC-SHA256(<DateRegionServiceKey>, "aws4_request")
```

**计算方法Note:**

> Signature = HMAC-SHA256(SigningKey, StringToSign)

4. Authorization header

```
AWS4-HMAC-SHA256 Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/s3/
aws4_request,SignedHeaders=content-encoding;content-length;host;x-amz-
content-sha256;x-amz-date;x-amz-decoded-content-length;x-amz-storage-
class,Signature=4f232c4386841ef735655705268965c44a0e4690baa4adea153f7db9fa80a0a9
```

5. Chunk 1: (65536 bytes, with value 97 for letter 'a')

   1. Chunk string to sign:
    ```
    AWS4-HMAC-SHA256-PAYLOAD
    20130524T000000Z
    20130524/us-east-1/s3/aws4_request
    4f232c4386841ef735655705268965c44a0e4690baa4adea153f7db9fa80a0a9
    e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    bf718b6f653bebc184e1479f1935b8da974d701b893afcf49e701f3e2f9f9c5a
    ```

    最后三行：previous-signature, hash(""), and hash(current-chunk-data)  

   2. Chunk signature:
   
    ```
    ad80c730a21e5b8d04586a2213dd63b9a0e99e0e2307b0ade35a65485a288648
    ```
   
   3. Chunk data sent:
    
    ```
    10000;chunk-
    signature=ad80c730a21e5b8d04586a2213dd63b9a0e99e0e2307b0ade35a65485a288648
    <65536-bytes>
    ```
5. Chunk 2: (1024 bytes, with value 97 for letter 'a')

   1. Chunk string to sign:
    ```
    AWS4-HMAC-SHA256-PAYLOAD
    20130524T000000Z
    20130524/us-east-1/s3/aws4_request
    ad80c730a21e5b8d04586a2213dd63b9a0e99e0e2307b0ade35a65485a288648
    e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    2edc986847e209b4016e141a6dc8716d3207350f416969382d431539bf292e4a
    ```

    最后三行：previous-signature, hash(""), and hash(current-chunk-data)  

   2. Chunk signature:
   
    ```
    0055627c9e194cb4542bae2aa5492e3c1575bbb81b612b7d234b86a503ef5497
    ```
   
   3. Chunk data sent:
    
    ```
    400;chunk-
    signature=0055627c9e194cb4542bae2aa5492e3c1575bbb81b612b7d234b86a503ef5497
    <1024 bytes>
    ```

6. Chunk 3: (0 byte)

   1. Chunk string to sign:
    
    ```
    AWS4-HMAC-SHA256-PAYLOAD
    20130524T000000Z
    20130524/us-east-1/s3/aws4_request
    0055627c9e194cb4542bae2aa5492e3c1575bbb81b612b7d234b86a503ef5497
    e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
    ```

    最后三行：previous-signature, hash(""), and hash(current-chunk-data)  

   2. Chunk signature:
   
    ```
    b6c6ea8a5354eaf15b3cb7646744f4275b71ea724fed81ceb9323e279d449df9
    ```
   
   3. Chunk data sent:
    
    ```
    0;chunk-signature=b6c6ea8a5354eaf15b3cb7646744f4275b71ea724fed81ceb9323e279d449df9
    ```


