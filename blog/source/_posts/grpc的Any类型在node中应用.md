---
uuid: 4218e420-c658-11e9-aaae-f32829255914
author: jiandandkl
email: emolingzhu@126.com
title: grpc的Any类型在node中应用
date: 2019-08-24 18:16:37
github: https://github.com/jiandandkl
avatar: https://avatars2.githubusercontent.com/u/16009933?v=4
tags: grpc node
---

 > 关于grpc就不介绍了,网上资料还是蛮多的,但是关于在node中的应用就比较少了,也不太友好

1. 没有google.protobuf.Any类型
 无需编译proto文件,引用grpc和@grpc/proto-loader即可
 
```javascript
const grpc = require('grpc')
const protoLoader = require('@grpc/proto-loader')
const path = require('path')

function createClient(
  { protoPath, packageName, serviceName, options },
  address,
  creds = grpc.credentials.createInsecure()
) {
  const pkgDef = grpc.loadPackageDefinition(protoLoader.loadSync(protoPath, options))
  const proto = getProtoFromPackageDefinition(pkgDef, packageName)
  return new proto[serviceName](address, creds)
}

function getProtoFromPackageDefinition(packageDefinition, packageName) {
  const pathArr = packageName.split('.')
  return pathArr.reduce((obj, key) => (obj && obj[key] !== 'undefined') ? obj[key] : undefined, packageDefinition)
}

const client = createClient({
  protoPath: path.resolve(__dirname, './xxx.proto'),
  // proto中package字段
  packageName: 'packageName',
  // proto中service方法
  serviceName: 'serviceName',
}, '10.0.0.0')

client.Report({
  // proto中定义的各字段
  type: 'HTTP',
  project: 'XXX',
  user_ip: '1111',
  takes: 123,
}, err => {
  if (err) {
    console.log(err)
  }
})
```

2. 某个字段为google.protobuf.Any类型

> `@grpc/proto-loader`已不支持,需要先将proto文件编译,使用的是[grpc\_tools\_node\_protoc\_ts](https://www.npmjs.com/package/grpc_tools_node_protoc_ts) (也能编译d.ts文件)

```javascript
const grpc = require('grpc')
const googleProtobufAnyPb = require('google-protobuf/google/protobuf/any_pb.js')
const services = require('./proto/hello_grpc_pb.js')
const messages = require('./proto/hello_pb.js')

const client = new services.ReportServiceClient(ip, grpc.credentials.createInsecure())

const request = new messages.Request()
// 可设置到Any类型extra的message
const requestHttp = new messages.HttpProto()
const anyPb = new googleProtobufAnyPb.Any()
// 获取message中的key
const keysHttp = requestHttp.toObject()

function grpcSend(data) {
    const { type } = data
    // typeUrl为Any类型对应的url,类似http中请求的url,规则为package + 对应的message
    let typeUrlAny = 'hello.HttpProto'
      
    Object.keys(data).forEach(key => {
        // 现有proto中extra字段为Any类型
        if (key === 'extra') {
         Object.keys(data.extra).forEach(keyAny => {
           if (keyAny in keysHttp) {
             // 先赋值requestHttp
             requestHttp[`set${upperCase(keyAny)}`](data.extra[keyAny])
           }
         })
         // 再将requestHttp序列化,并设置对应的typeUrl给anyPb
         anyPb.pack(requestHttp.serializeBinary(), typeUrlAny)
        } else {
         // 其他非Any类型,直接使用编译后文件中的如request.setName
         // 若为Type枚举类型,传对应的数字即可
         request[`set${upperCase(key)}`](data[key])
        }
    })
    // 最后将anyPb赋值给request
    request.setExtra(anyPb)
    // 调用rpc方法发送数据
    client.report(request, (err, res) => {
        if (err) {
          console.log('grpc error', err)
        }
  })
}
```
在github上传了一个抽取出来较为通用的示例(方便现有extra字段的扩展),仅供参考,[grpc-any-node-demo](https://github.com/jiandandkl/grpc-any-node-demo)