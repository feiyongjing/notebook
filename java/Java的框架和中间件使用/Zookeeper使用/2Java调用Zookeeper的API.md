# Zookeeper的Java客户端
- Zookeeper： Zookeeper是官方提供的原生java客户端，功能最全但是比较难以使用
- Zkclient： 停止维护，不推荐使用
- Curator： 封装了原生 ZooKeeper API 的痛点（如自动重连、分布式锁、选主、计数器等），适合需要实现复杂分布式功能的场景

## Curator使用
1.  