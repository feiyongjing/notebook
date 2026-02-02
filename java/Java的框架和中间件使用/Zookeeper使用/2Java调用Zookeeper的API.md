# Zookeeper的Java客户端
- Zookeeper： Zookeeper是官方提供的原生java客户端，功能最全但是比较难以使用
- Zkclient： 停止维护，不推荐使用
- Curator： 封装了原生 ZooKeeper API 的痛点（如自动重连、分布式锁、选主、计数器等），适合需要实现复杂分布式功能的场景

## Curator使用
1. 选择与Zookeeper服务端兼容的Curator引入依赖(别忘了spring-boot-starter-parent和spring-boot-starter-web)
~~~xml
<!-- curator核心依赖-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>5.6.0</version>
</dependency>
<!-- 高级功能（分布式锁、选主、计数器等，推荐）  -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>5.6.0</version>
</dependency>
~~~

2. 配置文件添加配置
~~~yml
zk:
  address: 127.0.0.1:2181
  sessionTimeoutMs: 60000
  connectionTimeoutMs: 15000
~~~

3. 连接配置设置
~~~java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ZKClientConfig {

    @Value("${zk.address}")
    String address;

    @Value("${zk.socketTimeout}")
    Integer sessionTimeoutMs;

    @Value("${zk.socketTimeout}")
    Integer connectionTimeoutMs;

    @Bean(destroyMethod = "close")
    public CuratorFramework a(){
        // connectString：指的是需要连接的zookeeperAddress ip和端口
        // sessionTimeoutMs：指的是会话超时时间
        // connectionTimeoutMs：指的是连接超时时间
        // retryPolicy：指的是连接zk时使用哪种重试策略，Curator默认提供了以下几种：
        //              RetryNTimes：重试N次
        //              ExponentialBackoffRetry：重试一定次数，每次重试时间依次递增
        //              RetryOneTime：重试一次
        //              RetryUntilElapsed：重试一定时间

        // 每次重试之间等待的初始时间3秒，最多重试10次
        RetryPolicy retry = new ExponentialBackoffRetry(3000, 10);
        CuratorFramework client = CuratorFrameworkFactory.newClient(address, sessionTimeoutMs, connectionTimeoutMs, retry);
        // 启动客户端连接
        client.start();

        return client;
    }
}
~~~

4. Znode节点操作
~~~java
import lombok.RequiredArgsConstructor;
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.data.Stat;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import org.springframework.util.StringUtils;

import java.util.ArrayList;
import java.util.List;

@Service
@RequiredArgsConstructor
public class ZnodeService {
    private final CuratorFramework zkClient;


    /**
     * 检查节点是否存在
     * @param nodePath 节点路径
     * @return 节点是否存在
     * @throws Exception
     */
    public Boolean checkExists(String nodePath) throws Exception {
        // 如果节点存在则返回的Stat对象中有该节点的的详细信息，与在客户端执行 get {nodePath} 命令看到信息一致
        Stat exists = zkClient.checkExists().forPath(nodePath);
        if (null != exists) {
            System.out.println("节点【" + nodePath + "】已存在");
        } else {
            System.out.println("节点【" + nodePath + "】不存在");
        }
        return null != exists;
    }

    /**
     * 创建节点，可以多级节点创建
     * @param mode 创建的Znode节点类型
     *             CreateMode.PERSISTENT              默认持久的Znode节点
     *             CreateMode.PERSISTENT_SEQUENTIAL   持久顺序性的Znode节点
     *             CreateMode.EPHEMERAL               临时的Znode节点
     *             CreateMode.EPHEMERAL_SEQUENTIAL    临时顺序性的Znode节点
     * @param nodePath 节点路径
     * @param data 节点数据
     * @return 是否创建成功
     * @throws Exception
     */
    public Boolean createNode(CreateMode mode, String nodePath, String data) throws Exception {
        // 创建节点，默认就是持久化的，并且如果创建节点时没有数据会默认将当前客户端的ip当成节点数据存储
        // creatingParentsIfNeeded 允许父节点不存在时先递归创建父节点然后创建子节点
        String path = "";
        if(StringUtils.hasLength(data)){
            path = zkClient.create()
                    .creatingParentsIfNeeded()
                    .withMode(mode)
                    .forPath(nodePath, data.getBytes());
        }else {
            path = zkClient.create()
                    .creatingParentsIfNeeded()
                    .withMode(mode)
                    .forPath(nodePath);
        }
        return path.startsWith(nodePath);
    }


    /**
     * 查询节点数据和元数据
     * @param nodePath 节点路径
     * @return 节点存储的数据
     * @throws Exception
     */
    public String getNode(String nodePath) throws Exception {
        // 直接获取某个节点数据
        //byte[] bytes = zkClient.getData().forPath(nodePath);

        //读取节点的存储的数据和元数据，并将元数据放到Stat中
        Stat stat1 = new Stat();
        byte[] bytes2 = zkClient.getData().storingStatIn(stat1).forPath(nodePath);
        String data = new String(bytes2);
        System.out.println("节点：【" + nodePath + "】，数据：" + data);
        System.out.println(stat1);
        return data;
    }


    /**
     * 查询某个节点下的所有子节点
     * @param nodePath 节点路径
     * @return 子节点列表
     * @throws Exception
     */
    public List<String> getSubNodeList(String nodePath) throws Exception {
        List<String> subNodeList = zkClient.getChildren().forPath(nodePath);
        if (CollectionUtils.isEmpty(subNodeList)) {
            return new ArrayList<>();
        }
        //遍历节点
        subNodeList.forEach(System.out::println);
        return subNodeList;
    }


    /**
     * 更新节点数据
     * @param nodePath 节点路径
     * @param data 更新的节点数据
     * @return 节点的元数据
     * @throws Exception
     */
    public Stat updateNode(String nodePath, String data) throws Exception {
        // 更新节点
        Stat stat = zkClient.setData().forPath(nodePath, data.getBytes());

        //指定版本号，更新节点，更新的时候如果指定数据版本的话，那么需要和zookeeper中当前数据的版本要一致，-1表示匹配任何版本
        //Stat stat = client.setData().withVersion(-1).forPath(nodePath, data.getBytes());
        System.out.println(stat);

        //异步设置某个节点数据
        //Stat stat1 = zkClient.setData().inBackground().forPath(nodePath, data.getBytes());
        //System.out.println(stat1.toString());
        return stat;
    }

    /**
     * 删除节点
     * @param nodePath 节点路径
     * @throws Exception
     */
    public void deleteNode(String nodePath) throws Exception {
        //删除节点
        zkClient.delete().forPath(nodePath);

        //删除节点，即使出现网络故障，zookeeper也可以保证删除该节点，并且设置回调函数
        //zkClient.delete().guaranteed().inBackground(new BackgroundCallback(){
        //    @Override
        //    public void processResult(CuratorFramework client, CuratorEvent event) throws Exception {
        //        System.out.println("回调逻辑执行");
        //        System.out.println(event);
        //    }
        //}).forPath(nodePath);

        //级联删除节点（如果当前节点有子节点，子节点也可以一同删除）
        //zkClient.delete().deletingChildrenIfNeeded().forPath(nodePath);
    }
}

~~~

5. NodeCache类型的Watcher事件监听
~~~java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.data.Stat;

public class NodeCacheWatchService {

    public static void main(String[] args) throws Exception {
        RetryPolicy retry = new ExponentialBackoffRetry(3000, 10);
        CuratorFramework zkClient = CuratorFrameworkFactory.newClient("127.0.0.1:2181", 60000, 15000, retry);
        // 启动客户端连接
        zkClient.start();


        String workerPath = "/nodeCache";
        //检查节点是否存在，没有则创建
        Stat exists = zkClient.checkExists().forPath(workerPath);
        if (null == exists) {
            zkClient.create().creatingParentsIfNeeded().forPath(workerPath);
        }

        try {
            NodeCache nodeCache =
                    new NodeCache(zkClient, workerPath);
            NodeCacheListener l = new NodeCacheListener() {
                @Override
                public void nodeChanged() throws Exception {
                    String path = nodeCache.getPath();

                    ChildData childData = nodeCache.getCurrentData();
                    if(childData !=null) {
                        System.out.println("ZNode节点状态改变, path=" + childData.getPath());
                        System.out.println("ZNode节点状态改变, data=" + new String(childData.getData(), "Utf-8"));
                        System.out.println("ZNode节点状态改变, stat=" + childData.getStat());
                    }else {
                        System.out.println("ZNode节点被删除了, path=" + path);
                    }
                }
            };
            nodeCache.getListenable().addListener(l);
            nodeCache.start();

            // 第1次变更节点数据
            zkClient.setData().forPath(workerPath, "第1次更改内容".getBytes());
            Thread.sleep(1000);

            // 第2次变更节点数据
            zkClient.setData().forPath(workerPath, "第2次更改内容".getBytes());

            Thread.sleep(1000);

            // 第3次变更节点数据
            zkClient.setData().forPath(workerPath, "第3次更改内容".getBytes());
            Thread.sleep(1000);

            // 第4次变更删除节点
            zkClient.delete().forPath(workerPath);
            Thread.sleep(1000);

            // 第5次变更新增节点
            zkClient.create().creatingParentsIfNeeded().forPath(workerPath);
            Thread.sleep(Integer.MAX_VALUE);
        } catch (Exception e) {
            System.out.println("创建NodeCache监听失败, path=" + workerPath);
        }

    }
}
~~~

6. PathChildrenCache类型的Watcher事件监听
- 只能监听子节点，监听不到当前节点
- 不能递归监听，子节点下的子节点不能递归监控
~~~java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.PathChildrenCache;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheEvent;
import org.apache.curator.framework.recipes.cache.PathChildrenCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.data.Stat;

import java.io.UnsupportedEncodingException;

public class PathChildrenCacheWatchService {


    public static void main(String[] args) throws Exception {
        RetryPolicy retry = new ExponentialBackoffRetry(3000, 10);
        CuratorFramework zkClient = CuratorFrameworkFactory.newClient("127.0.0.1:2181", 60000, 15000, retry);
        // 启动客户端连接
        zkClient.start();


        String workerPath = "/pathChildrenCache";
        String subWorkerPath = workerPath + "/id";
        //检查节点是否存在，没有则创建
        Stat exists = zkClient.checkExists().forPath(workerPath);
        if (null == exists) {
            zkClient.create().creatingParentsIfNeeded().forPath(workerPath);
        }

        try {

            // cacheData表示是否把节点内容缓存起来。如果cacheData为true，那么接收到节点列表变更事件的同时，会将获得节点内容
            // executorService 和threadFactory参数差不多，表示通过传入的线程池或者线程工厂，来异步处理监听事件。
            // threadFactory参数（如果有）表示线程池工厂，当PathChildrenCache内部需要开启新的线程执行时，使用该线程池工厂来创建线程。
            PathChildrenCache cache =
                    new PathChildrenCache(zkClient, workerPath, true);
            PathChildrenCacheListener l =
                    new PathChildrenCacheListener() {
                        @Override
                        public void childEvent(CuratorFramework client,
                                               PathChildrenCacheEvent event){
                            try {
                                ChildData data = event.getData();
                                switch (event.getType()) {
                                    case CHILD_ADDED:
                                        System.out.println("子节点增加, path=" + data.getPath() + ", data=" + new String(data.getData(), "UTF-8"));
                                        break;
                                    case CHILD_UPDATED:
                                        System.out.println("子节点更新, path=" + data.getPath() + ", data=" + new String(data.getData(), "UTF-8"));
                                        break;
                                    case CHILD_REMOVED:
                                        System.out.println("子节点删除, path=" + data.getPath() + ", data=" + new String(data.getData(), "UTF-8"));
                                        break;
                                    default:
                                        break;
                                }

                            } catch (
                                    UnsupportedEncodingException e) {
                                e.printStackTrace();
                            }
                        }
                    };
            cache.getListenable().addListener(l);


            // 需要传入启动的模式。可以传入三种模式，也就是API列表中看到的StartMode，其中定义了下面三种枚举：
            // BUILD_INITIAL_CACHE：启动时，同步初始化cache，以及创建cache后，就从服务器拉取对应的数据。
            // POST_INITIALIZED_EVENT：启动时，异步初始化cache，初始化完成触发PathChildrenCacheEvent.Type#INITIALIZED事件，cache中Listener会收到该事件的通知。
            // NORMAL：启动时，异步初始化cache，完成后不会发出通知。
            cache.start(PathChildrenCache.StartMode.BUILD_INITIAL_CACHE);
            Thread.sleep(1000);

            for (int i = 0; i < 3; i++) {
                zkClient.create().creatingParentsIfNeeded().forPath(subWorkerPath + i);
                Thread.sleep(1000);
            }

            for (int i = 0; i < 3; i++) {
                String data = subWorkerPath + i + "内容被修改";
                zkClient.setData().forPath(subWorkerPath + i, data.getBytes());
                Thread.sleep(1000);
            }

            zkClient.delete().deletingChildrenIfNeeded().forPath(workerPath);

        } catch (Exception e) {
            System.out.println("PathCache监听失败, path=" + workerPath);
        }

    }

}
~~~

7. TreeCache类型的Watcher事件监听
~~~java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.TreeCache;
import org.apache.curator.framework.recipes.cache.TreeCacheEvent;
import org.apache.curator.framework.recipes.cache.TreeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.data.Stat;

import java.io.UnsupportedEncodingException;

public class TreeCacheWatchService {
    public static void main(String[] args) throws Exception {
        RetryPolicy retry = new ExponentialBackoffRetry(3000, 10);
        CuratorFramework zkClient = CuratorFrameworkFactory.newClient("127.0.0.1:2181", 60000, 15000, retry);
        // 启动客户端连接
        zkClient.start();


        String workerPath = "/treeCache";
        String subWorkerPath = workerPath + "/id";
        //检查节点是否存在，没有则创建
        Stat exists = zkClient.checkExists().forPath(workerPath);
        if (null == exists) {
            zkClient.create().creatingParentsIfNeeded().forPath(workerPath);
        }

        try {
            TreeCache treeCache =
                    new TreeCache(zkClient, workerPath);
            TreeCacheListener l =
                    new TreeCacheListener() {
                        @Override
                        public void childEvent(CuratorFramework client,
                                               TreeCacheEvent event) {
                            try {
                                ChildData data = event.getData();
                                if (data == null) {
                                    System.out.println("数据为空");
                                    return;
                                }
                                switch (event.getType()) {
                                    case NODE_ADDED:
                                        System.out.println("[TreeCache]节点增加, path=" + data.getPath() + ", data=" + new String(data.getData(), "UTF-8"));
                                        break;
                                    case NODE_UPDATED:
                                        System.out.println("[TreeCache]节点更新, path=" + data.getPath() + ", data=" + new String(data.getData(), "UTF-8"));
                                        break;
                                    case NODE_REMOVED:
                                        System.out.println("[TreeCache]节点删除, path=" + data.getPath() + ", data=" + new String(data.getData(), "UTF-8"));
                                        break;
                                    default:
                                        break;
                                }

                            } catch (
                                    UnsupportedEncodingException e) {
                                e.printStackTrace();
                            }
                        }
                    };
            treeCache.getListenable().addListener(l);
            treeCache.start();
            Thread.sleep(1000);
            for (int i = 0; i < 3; i++) {
                zkClient.create().creatingParentsIfNeeded().forPath(subWorkerPath + i);
                Thread.sleep(1000);
            }

            for (int i = 0; i < 3; i++) {
                String data = subWorkerPath + i + "内容被修改";
                zkClient.setData().forPath(subWorkerPath + i, data.getBytes());
                Thread.sleep(1000);
            }

            zkClient.delete().deletingChildrenIfNeeded().forPath(workerPath);
        } catch (Exception e) {
            System.out.println("PathCache监听失败, path=" + workerPath);
        }

    }

}
~~~

8. Curator实现分布式锁
在Curator中有五种锁方案:
- InterProcessSemaphoreMutex:分布式排它锁(非可重入锁)
- InterProcessMutex:分布式可重入排它锁
- InterProcessReadWriteLock:分布式读写锁
- InterProcessMultiLock:将多个锁作为单个实体管理的容器
- InterProcessSemaphoreV2:共享信号量
### InterProcessMutex:分布式可重入排它锁案例
~~~java
package com.mobil.ocm.admin.service;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;

public class InterprocessLock {

    static Integer tickets = 100;

    private static CuratorFramework getZkClient() {
        RetryPolicy retry = new ExponentialBackoffRetry(3000, 10);
        CuratorFramework zkClient = CuratorFrameworkFactory.newClient("127.0.0.1:2181", 60000, 15000, retry);
        zkClient.start();
        return zkClient;
    }

    public static void main(String[] args) throws InterruptedException {
        CuratorFramework zkClient1 = getZkClient();
        CuratorFramework zkClient2 = getZkClient();
        String lockPath = "/lock";

        // 模拟两个客户端，每个客户端都不停的抢购商品，直到商品抢购完
        Thread t1 = new Thread(new TestThread(zkClient1, lockPath), "客户1");
        Thread t2 = new Thread(new TestThread(zkClient2, lockPath), "客户2");

        t1.start();
        t2.start();

        // 修复：等待线程真正执行完毕再关闭客户端
        t1.join();
        t2.join();

        zkClient1.close();
        zkClient2.close();
        System.out.println("所有线程执行完毕，客户端已关闭");
    }

    static class TestThread implements Runnable {

        private final InterProcessMutex lock;

        public TestThread(CuratorFramework zkClient, String lockPath) {
            // 每个线程一把锁，避免上锁后被其他线程上锁和解锁
            this.lock = new InterProcessMutex(zkClient, lockPath);
        }

        @Override
        public void run() {

            while (true) {
                try {
                    // 尝试获取锁，最多等 3 秒
                    if (lock.acquire(3, java.util.concurrent.TimeUnit.SECONDS)) {

                        if (tickets > 0) {
                            tickets=tickets-1;
                            System.out.println(Thread.currentThread().getName() +
                                    " 成功抢购1件商品，还剩下 " + tickets + " 件");
                        } else {
                            System.out.println(Thread.currentThread().getName() +
                                    " 商品已售罄，抢购失败");
                            return;
                        }

                        Thread.sleep(100); // 模拟业务耗时
                    } else {
                        System.out.println(Thread.currentThread().getName() + " 获取锁超时，抢购失败");
                    }

                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    // 必须释放锁
                    try {
                        if (lock.isAcquiredInThisProcess()) {
                            lock.release();
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
~~~




