---
tags:
  - Node.js
  - gRPC
  - RPC
  - 微服务通信
date: 2026-05-12
status: 已完成
difficulty: 进阶
---

# RPC 与微服务通信

## What — 是什么

> RPC（Remote Procedure Call）让远程服务调用像本地函数调用一样简单。gRPC 是 Google 推出的高性能 RPC 框架，基于 Protocol Buffers 和 HTTP/2，是微服务间通信的首选方案。

**核心概念：**

- **gRPC**：基于 HTTP/2 + Protobuf 的高性能 RPC 框架，支持四种通信模式
- **Protocol Buffers**：二进制序列化格式，体积小（比 JSON 小 3-10 倍），速度快
- **四种模式**：Unary（一元/请求-响应）、Server Streaming（服务端流）、Client Streaming（客户端流）、Bidirectional Streaming（双向流）
- **服务发现**：服务注册与发现机制，动态获取服务实例地址
- **负载均衡**：客户端侧（gRPC 内置）或服务端侧（Nginx/Envoy）

**关键特性：**

- Protobuf 生成强类型客户端/服务端代码
- HTTP/2 多路复用，一个连接多个并发请求
- 内置健康检查、超时、重试等机制
- 流式通信适合实时数据推送

## Why — 为什么

**适用场景：**

- 微服务间高性能同步通信
- 需要强类型接口契约
- 实时数据流（行情/日志/监控）
- 多语言微服务架构

**对比通信方案：**

| 维度 | gRPC | REST/HTTP | 消息队列 |
|------|------|-----------|---------|
| 模式 | 同步 | 同步 | 异步 |
| 性能 | 极高 | 中 | 中 |
| 类型安全 | 强（Protobuf） | 弱（OpenAPI） | 无 |
| 浏览器支持 | 需 gRPC-Web | 原生 | 不适用 |
| 适用场景 | 微服务间 | 对外API | 异步解耦 |

## How — 怎么用

### 代码示例

```protobuf
// user.proto
syntax = "proto3";
package user;
service UserService {
    rpc GetUser(GetUserRequest) returns (User);
    rpc ListUsers(ListUsersRequest) returns (stream User);
}
message GetUserRequest { int32 id = 1; }
message ListUsersRequest { int32 limit = 1; }
message User { int32 id = 1; string name = 2; string email = 3; }
```

```javascript
// 服务端
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDef = protoLoader.loadSync('user.proto');
const userProto = grpc.loadPackageDefinition(packageDef).user;

const server = new grpc.Server();
server.addService(userProto.UserService.service, {
    GetUser: async (call, callback) => {
        const user = await db.users.findById(call.request.id);
        callback(null, user);
    },
    ListUsers: async (call) => {
        const users = await db.users.findMany(call.request.limit);
        for (const user of users) call.write(user);
        call.end();
    }
});
server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => server.start());

// 客户端
const client = new userProto.UserService('localhost:50051', grpc.credentials.createInsecure());
client.GetUser({ id: 1 }, (err, user) => console.log(user));
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 浏览器不能直连 gRPC | HTTP/2 需要特殊支持 | gRPC-Web 或 gRPC-Gateway |
| Protobuf 改更影响大 | 字段编号不能改 | 新增字段用新编号，旧编号永不复用 |
| 服务发现缺失 | 硬编码地址 | Consul/Nacos 服务注册发现 |

### 最佳实践

- Protobuf 字段编号一旦分配不可更改
- 服务间通信用 gRPC，对外 API 用 REST（gRPC-Gateway 桥接）
- 设置合理的超时和重试策略
- 使用健康检查协议

## 面试题

**Q1: gRPC 为什么比 REST 快？**
> 三个原因：① Protobuf 二进制序列化比 JSON 文本序列化快 3-10 倍，体积小 3-10 倍；② HTTP/2 多路复用，一个 TCP 连接并行多个请求（HTTP/1.1 需要多个连接或排队）；③ 强类型编译期生成代码，无需运行时反射和解析。实测：gRPC Unary 吞吐量约 REST 的 5-10 倍。

**Q2: gRPC 四种通信模式的使用场景？**
> ① Unary——普通请求-响应，类似 REST（查用户/创建订单）；② Server Streaming——客户端一次请求，服务端持续推送（实时行情/日志流/监控数据）；③ Client Streaming——客户端持续发送，服务端一次响应（文件上传/批量数据导入）；④ Bidirectional Streaming——双方持续收发（聊天/实时协作/游戏状态同步）。

---

**相关链接：**
- [[微服务架构]]
- [[消息队列与事件驱动]]
- [[RESTful API设计]]
- gRPC 文档：https://grpc.io/docs/
