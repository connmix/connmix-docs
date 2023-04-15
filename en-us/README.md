## Introduction

CONNMIX is a distributed long-connection gateway based on Go and Lua, which can quickly develop high-performance distributed Socket and WebSocket long-connection services. It allows Lua programming like openresty (ngx_lua) and can be quickly integrated into existing systems developed in any programming language such as Java, PHP, Go, Node.js, Python, and C#.

The value of the long-connection gateway lies in its encapsulation of Socket and WebSocket communication details and decoupling from the business system, making the long-connection gateway and the business system independently optimized and iterated. Therefore, users only need to focus on the development of the business system, which can significantly shorten the development cycle. Second, the gateway provides simple and easy-to-use HTTP and WS push channels, supports multiple programming languages for access, and is convenient for system integration and use. In addition, the gateway adopts a distributed architecture, which can achieve horizontal scaling, load balancing, and high availability of services. Finally, the gateway integrates monitoring and alerts to provide early warning when the system is abnormal, ensuring the health and stability of the service.

## Use Scenarios

- CONNMIX is used to quickly develop distributed long-connection services, such as Internet, instant messaging, APP development, online games, hardware communication, smart home, Internet of Things, and other fields.
- CONNMIX uses Lua programming, so it can customize network communication protocols while having hot-update features, making it very suitable for online games, IoT, and other fields.

## Project Features

- Developed based on Go, providing natural high concurrency support and extreme multithreading utilization.
- Built-in Lua VM, allowing you to write business logic and customize any network communication protocols. Meanwhile, the official provides most of the commonly used protocols.
- Automated service mesh, distributed execution. Cluster expansion just requires starting a new node.
- Quickly integrate with existing systems developed in any programming language such as Java, PHP, Go, Node.js, Python, and C#.
- Hot update of configuration, communication protocol hot update, and connection continuity when restarting the service.
- Debug-friendly: Lua protocol debugging information is directly printed in stdout, making development and debugging very convenient.
- Truly zero dependency, no need to install any third-party middleware.
- No need to compile, just a binary and related configuration files. Download the compressed package and start using it.
