---
title: "Willin: Azure Node.js Blob文件上传"
date: 2017-06-07 18:33:12
categories: Dev
tags: [node.js, azure, storage, oss]
---

{% cq %}
对官方文档一些需要额外注意的细节整理
{% endcq %}

azure-storage官方文档: <http://azure.github.io/azure-storage-node/>

## 建立连接

有3种方式(文档中未提及):

<!--more-->

### 1. 通过环境变量

```bash
AZURE_STORAGE_CONNECTION_STRING="valid storage connection string" node app.js
```

应用程序内:

```js
const azure = require('azure-storage');
const blobService = azure.createBlobService();
// code here
```

### 2.连接字符串

```js
const azure = require('azure-storage');
const blobService = azure.createBlobService('connectionString'); // 类似: DefaultEndpointsProtocol=https;AccountName=*****;AccountKey=*****;EndpointSuffix=*****.core.chinacloudapi.cn
// code here
```

### 3.账号+密钥

```js
const azure = require('azure-storage');
const blobService = azure.createBlobService('storageAccount', 'storageAccessKey', 'storageHost'); 
// code here
```

## 上传示例

因为POST请求接收到的大部分是Stream.所以采用Sream的方式上传.

```js
// azure.js
const azure = require('azure-storage');
const { getDefer } = require('@dwing/common');

const blobService = azure.createBlobService('accountName', 'accessKey', 'host');

exports.createBlockBlobFromStream = (container, filename, blob) => {
  const deferred = getDefer();
  blob.on('error', (err) => {
    deferred.reject(err);
  });
  blob.pipe(blobService.createWriteStreamToBlockBlob(container, filename));
  blob.on('end', () => {
    deferred.resolve(1);
  });
  return deferred.promise;
};
```

测试代码:

```js
// demo.js
const { createBlockBlobFromStream } = require('./azure');
const fs = require('fs');
const path = require('path');

const stream = fs.createReadStream(path.join(__dirname, '/testfile'));

(async () => {
  const result = await createBlockBlobFromStream('container', 'filename', stream);
  console.log(result);
})();
```
