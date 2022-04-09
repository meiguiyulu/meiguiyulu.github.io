# 																			Zookeeper

****

## 一、Zookeeper介绍

### 1. 什么是Zookeeper

​		ZooKeeper 是⼀种分布式协调服务，⽤于管理⼤型主机。在分布式环境中协调和管理服务是⼀个复杂的过程。ZooKeeper 通过其简单的架构和 API 解决了这个问题。ZooKeeper 允许开发⼈员专注于核⼼应⽤程序逻辑，⽽不必担⼼应⽤程序的分布式特性。

### 2. Zookeeper的应用场景

- 分布式协调组件

![image-20220407122814639](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407122814639.png)

在分布式系统中，需要有 zookeeper 作为分布式协调组件，协调分布式系统中的状态。

- 分布式锁

zk在实现分布式锁上，可以做到强⼀致性，关于分布式锁相关的知识，在之后的ZAB协议中介绍。

- 无状态化实现

![image-20220407122939960](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407122939960.png)

### 

## 二、搭建Zookeeper服务器

### 1. 安装

- [官网](https://zookeeper.apache.org/index.html)
  - ![image-20220407142607443](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407142607443.png)
  - ![image-20220407142625410](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407142625410.png)
- 将 zookeeper 目录 conf 文件夹下的 `zoo_sample.cfg` 改名为 `zoo.cfg`
- 若提示 `JAVA_HOME is not set`问题
  - ![image-20220407142841965](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407142841965.png)

### 2. zoo.cfg 配置⽂件说明

```bash
# zookeeper时间配置中的基本单位 (毫秒)
tickTime=2000
# 允许follower初始化连接到leader最⼤时⻓，它表示tickTime时间倍数，即:initLimit*tickTime
initLimit=10
# 允许follower与leader数据同步最⼤时⻓,它表示tickTime时间倍数
syncLimit=5
# zookeper 数据存储⽬录及⽇志保存⽬录（如果没有指明dataLogDir，则⽇志也保存在这个⽂件中）
dataDir=/tmp/zookeeper
# 对客户端提供的端⼝号
clientPort=2181
# 单个客户端与zookeeper最⼤并发连接数
maxClientCnxns=60
# 保存的数据快照数量，之外的将会被清除
autopurge.snapRetainCount=3
# ⾃动触发清除任务时间间隔，⼩时为单位。默认为0，表示不⾃动清除。
autopurge.purgeInterval=1
```

### 3. Zookeeper服务器的操作命令(Linux)

- 启动 zk 服务器：

  - ```sh
    ./bin/zkServer.sh start ./conf/zoo.cfg
    ```

- 查看 zk 服务器状态：

  - ```sh
    ./bin/zkServer.sh status ./conf/zoo.cfg
    ```

- 停⽌ zk 服务器：

  - ```sh
    ./bin/zkServer.sh stop ./conf/zoo.cfg
    ```

## 三、Zookeeper内部的数据模型

### 1.  zookeeper是如何保存数据的

 `zookeeper` 中的数据是保存在节点上的，节点就是 `znode`，多个 `znode`之间构成⼀颗树的⽬录结构。

`Zookeeper` 的数据模型很像数据结构当中的树，也很像⽂件系统的⽬录。

![image-20220407173046810](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407173046810.png)

树是由节点所组成，**`Zookeeper` 的数据存储也同样是基于节点**，这种节点叫做 **`Znode`**。

但是，不同于树的节点，**`Znode` 的引⽤⽅式是路径引⽤，**类似于⽂件路径：

```bash
[zk: localhost:2181(CONNECTED) 1] create /test1
Created /test1
[zk: localhost:2181(CONNECTED) 2] ls /
[test1, zookeeper]
[zk: localhost:2181(CONNECTED) 3] create /test1/test11  # 创建节点
Created /test1/test11
[zk: localhost:2181(CONNECTED) 5] ls /
[test1, zookeeper]
[zk: localhost:2181(CONNECTED) 6] create /test2 abc     # 创建数据
Created /test2
[zk: localhost:2181(CONNECTED) 7] get /test2
abc
[zk: localhost:2181(CONNECTED) 8] get test2
Path must start with / character   # 不加 / 就会报这种错误
[zk: localhost:2181(CONNECTED) 9]
```

这样的层级结构，让每⼀个 `Znode` 节点拥有唯⼀的路径，就像命名空间⼀样对不同信息作出清晰的隔离。

### 2. znode结构

`Zookeeper` 中的 `znode`，包含了四个部分：

- `data`：保存数据
- `acl`：权限，定义了什么样的⽤户能够操作这个节点，且能够进⾏怎样的操作。
  - `c: create` 创建权限，允许在该节点下创建⼦节点
  - `w：write` 更新权限，允许更新该节点的数据
  - `r：read` 读取权限，允许读取该节点的内容以及⼦节点的列表信息
  - `d：delete` 删除权限，允许删除该节点的⼦节点
  - `a：admin` 管理者权限，允许对该节点进⾏ `acl` 权限设置
- `stat`：描述当前 `znode` 的元数据
- `child`：当前节点的⼦节点

### 3. zookeeper中节点znode的类型

- 持久节点：创建出的节点，在会话结束后仍然存在。保存数据。
- 持久序号节点：创建出的节点。根据先后顺序，会在节点之后带上一个数值，越往后执行数值越大，适用于分布式锁的应用场景。单调递增。 `create -s`
- 临时节点：

临时节点是指在会话结束后自动删除的节点。通过这个特性，`zookeeper`可以实现**服务注册与发现**的效果。 `create -e`

![image-20220407193057671](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407193057671.png)

- 临时序号节点：跟持久序号节点相同，适用于临时的分布式锁。`create -e -s`
- Container节点：当Container容器节点中没有任何子节点的时候，该节点会被 `zookeeper` 定期删除(60s)。
- TTL节点：可以指定节点的到期时间，到期后被 `zookeeper` 定时删除。只能通过系统配置 `zookeeper.extendedTypesEnabled=true` 开启。

### 4. zookeeper的数据持久化

`zookeeper` 的数据是运行在内存中，提供了两种持久化机制：

- 事务日志

`zookeeper` 把执行的命令以日志形式保存在 `dataLogDir` 指定的路径中的文件中( 若没有指定`dataLogDir` ，则按照`dataDir`的路径)

- 数据快照

`zookeeper`会在一定的时间间隔内做一次内存数据的快照，把该时刻的内存数据保存在快照文件中。

> **`zookeeper`通过两种形式的持久化，在恢复时先恢复快照文件中的数据到内存中，再用日志文件中的数据做增量恢复，这样的恢复速度很快。**

## 四、Zookeeper客户端(zkCli)的使⽤

### 1. 多节点类型创建

- 创建持久节点            `create`
- 创建持久序号节点    `create -s`
- 创建临时节点            `create -e`
- 创建临时序号节点    `create -e -s`
- 创建容器节点            `create -c`

### 2. 查询节点

- 普通查询                   `ls`    `ls -R` 表示递归查询 子节点

  - ```markdown
    [zk: localhost:2181(CONNECTED) 10] create /test1/test11/test111
    Created /test1/test11/test111
    [zk: localhost:2181(CONNECTED) 11] ls /
    [test1, test2, zookeeper]
    [zk: localhost:2181(CONNECTED) 12] ls -R /test1
    /test1
    /test1/test11
    /test1/test11/test111
    [zk: localhost:2181(CONNECTED) 13]
    ```

- 查询节点相关信息   `get(-s)` `-s` 表示详细查询

  - `cZxid`: 创建节点的事务ID

  - `mZxid`：修改节点的事务ID

  - `pZxid`：添加和删除⼦节点的事务ID

  - `ctime`：节点创建的时间

  - `mtime`: 节点最近修改的时间

  - `dataVersion`: 节点内数据的版本，每更新⼀次数据，版本会+1

  - `aclVersion`: 此节点的权限版本

  - `ephemeralOwner`: 如果当前节点是临时节点，该值是当前节点所有者的session

  - `id`: 如果节点不是临时节点，则该值为零。

  - `dataLength`: 节点内数据的⻓度

  - `numChildren`: 该节点的⼦节点个数

  - 例如

    - ```markdown
      [zk: localhost:2181(CONNECTED) 6] create /test2 abc
      Created /test2
      [zk: localhost:2181(CONNECTED) 7] get /test2
      abc
      [zk: localhost:2181(CONNECTED) 9] get -s /test2
      abc
      cZxid = 0x4
      ctime = Thu Apr 07 17:39:42 CST 2022
      mZxid = 0x4
      mtime = Thu Apr 07 17:39:42 CST 2022
      pZxid = 0x4
      cversion = 0
      dataVersion = 0
      aclVersion = 0
      ephemeralOwner = 0x0
      dataLength = 3
      numChildren = 0
      ```

### 3. 删除节点

- 普通删除

  - `delete/deleteall`

  - ```markdown
    [zk: localhost:2181(CONNECTED) 13] ls
    ls [-s] [-w] [-R] path
    [zk: localhost:2181(CONNECTED) 14] ls /
    [test1, test2, zookeeper]
    [zk: localhost:2181(CONNECTED) 15] delete test2
    Path must start with / character
    [zk: localhost:2181(CONNECTED) 16] delete /test2
    [zk: localhost:2181(CONNECTED) 17] ls /
    [test1, zookeeper]
    [zk: localhost:2181(CONNECTED) 18] delete /test1
    Node not empty: /test1
    [zk: localhost:2181(CONNECTED) 19] deleteall /test1
    [zk: localhost:2181(CONNECTED) 20] ls /
    [zookeeper]
    [zk: localhost:2181(CONNECTED) 21]
    ```

- 乐观锁删除

  - `delete -v 数字 节点名称`

  - ```java
    [zk: localhost:2181(CONNECTED) 21] create /test1
    Created /test1
    [zk: localhost:2181(CONNECTED) 22] delete -v 1 /test1
    version No is not valid : /test1
    [zk: localhost:2181(CONNECTED) 23] delete -v 0 /test1
    [zk: localhost:2181(CONNECTED) 24]
    ```

### 4. 权限设置

- 设置当前会话的账号和密码

  ```bash
  addauth digest 账户:密码
  例如：
  addauth digest xiaoming:123456
  ```

- 创建节点并设置权限

  ```bash
  create /节点名 abcd auth:账户:密码:权限
  例如：
  create /test-node abcd auth:xiaowang:123456:cdwra
  ```

- 在另⼀个会话中必须先使⽤账号密码，才能拥有操作该节点的权限

## 五、Java操作Zookeeper

### 1. 引入依赖

```xml
        <!--zkclient-->
        <!-- https://mvnrepository.com/artifact/com.101tec/zkclient -->
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.11</version>
        </dependency>
```

### 2. 连接

```java
    public static void main(String[] args) {

        // 获取连接
        /**
         * 参数1：服务器的ip地址:端口号
         * 参数2：会话超时时间
         * 参数3：连接超时时间
         * 参数4: 序列化方式
         */
        ZkClient zkClient = new ZkClient(
                "127.0.0.1:2181",
                6000 * 30,
                6000,
                new SerializableSerializer()
        );

        System.out.println(zkClient);

        // 释放资源
        zkClient.close();

    }
```

### 3. 创建节点

```java
    @Test // 在zookeeper上创建节点
    public void testCreateNode() {
        // 1. 持久节点
        zkClient.create("/node1", "持久节点", CreateMode.PERSISTENT);
        // 2. 持久顺序(序号)节点
        zkClient.create("/node2", "持久顺序(序号)节点", CreateMode.PERSISTENT_SEQUENTIAL);
        // 3. 临时节点
        zkClient.create("/node3", "临时节点", CreateMode.EPHEMERAL);
        // 4. 临时顺序(序号)节点
        zkClient.create("/node4", "临时顺序(序号)节点", CreateMode.EPHEMERAL_SEQUENTIAL);
    }
```

### 4. 删除节点

```java
    // 删除节点
    @Test
    public void testDeleteNode() {
        // 删除没有子节点的节点   返回值：是否删除成功
        boolean delete = zkClient.delete("/node1");

        // 乐观锁删除节点, 1表示期望的version  返回值：是否删除成功
        boolean delete1 = zkClient.delete("/node3", 1);

        // 递归删除节点信息 返回值：是否删除成功
        boolean deleteRecursive = zkClient.deleteRecursive("/node2");
    }
```

### 5. 查询当前节点下所有子节点

```java
    // 查询当前节点下所有子节点
    @Test
    public void testFindNode() {
        // 获取指定节点下的节点信息     返回值：当前节点的子节点名称
        List<String> children = zkClient.getChildren("/");
        for (String child : children) {
            System.out.println(child);
        }
    }
```

### 6. 查看节点信息

```java
    // 查看某个节点的数据 注意：通过java客户端操作需要保障节点存储时数据序列化和获取节点时数据序列化方式必须一致
    @Test
    public void testReadData() {
        Object data = zkClient.readData("/node1");
        System.out.println(data);
    }
```

### 7. 获取节点的状态信息

```java
    // 查看节点的数据并且获取状态信息
    @Test
    public void testFindNodeState(){
        Stat stat = new Stat();
        Object data = zkClient.readData("/node1", stat);
        System.out.println(data);
        System.out.println(stat);
    }
```



![image-20220408112749678](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408112749678.png)



### 8. 修改节点数据

```java
    // 修改节点信息
    @Test
    public void testUpdateData() {
        zkClient.writeData("/node1", new User(1, "name", 18, new Date()));
    }
```

### 9. 监听节点数据变化

```java
    // 监听节点数据的变化
    @Test
    public void testOnNodeDataChange() throws IOException {
        zkClient.subscribeDataChanges("/node1", new IZkDataListener() {
            // 当节点的值修改时，会自动调用这个方法   将修改后的节点的名字和变化之后的数据传递给该方法
            @Override
            public void handleDataChange(String dataPath, Object data) throws Exception {
                System.out.println("当前节点的路径：" + dataPath);
                System.out.println("变化后的数据： " + data);
            }

            // 当节点的值被删除的时候，会自动调用这个方法   将节点的名字用参数的形式传递给该方法
            @Override
            public void handleDataDeleted(String dataPath) throws Exception {
                System.out.println("当前节点的路径：" + dataPath);
            }
        });
        System.in.read(); // 阻塞当前监听
    }
```

### 10. 监听节点目录变化

```java
    // 监听节点目录的变化
    @Test
    public void testOnNodeChange() throws IOException {
        // 当节点的目录发送变化时，会自动调用这个方法
        // 参数1：父节点名称
        // 参数2：父节点的所有子节点名称
        zkClient.subscribeChildChanges("/node1", new IZkChildListener() {
            @Override
            public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
                System.out.println("父节点名称： " + parentPath);
                System.out.println("发生变更后孩子节点名称： ");
                for (String child : currentChilds) {
                    System.out.println(child);
                }
            }
        });
        System.in.read();
    }
```

## 六、Curator客户端的使用

### 1. Curator介绍

​		Curator是Netflix公司开源的⼀套zookeeper客户端框架，Curator是对Zookeeper⽀持最好的客户端框架。Curator封装了⼤部分Zookeeper的功能，⽐如Leader选举、分布式锁等，减少了技术⼈员在使⽤Zookeeper时的底层细节开发⼯作。

官网：https://curator.apache.org/。

### 2. 引入Curator

- 依赖

  ```xml
          <!--Curator-->
          <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-framework</artifactId>
              <version>5.2.1</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-recipes</artifactId>
              <version>5.2.1</version>
          </dependency>
          <!--Zookeeper-->
          <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
          <dependency>
              <groupId>org.apache.zookeeper</groupId>
              <artifactId>zookeeper</artifactId>
              <version>3.8.0</version>
          </dependency>
      </dependencies>
  ```

- 配置类

  - ```java
    /**
    newClient()参数如下：
    curator:
      retryCount: 5
      elapsedTimeMs: 5000
      connectString: 127.0.0.1:2181
      sessionTimeoutMs: 60000
      connectionTimeoutMs: 5000
    */
    
    @Configuration
    public class CuratorConfig {
    
        @Autowired
        WrapperZK wrapperZK;
    
        @Bean(initMethod = "start")
        public CuratorFramework curatorFramework() {
            return CuratorFrameworkFactory.newClient(
                    wrapperZK.getConnectString(), // 127.0.0.1:2181
                    wrapperZK.getSessionTimeoutMs(), // 60000
                    wrapperZK.getConnectionTimeoutMs(), // 5000
                    new RetryNTimes(wrapperZK.getRetryCount(), wrapperZK.getElapsedTimeMs())
            );
        }
    }
    ```

### 3. 创建节点

```java
    @Autowired
    CuratorFramework curatorFramework;

    @Test
    void contextLoads() {
    }

    @Test
    void createNode() throws Exception {
        // 持久节点
        String path = curatorFramework.create().forPath("/curator-node");
        //添加临时序号节点
        String path1 =
                curatorFramework.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath("/curator-EPHEMERAL_SEQUENTIAL-node", "some-data".getBytes());
        System.out.println(String.format("curator create node :%s successfully.", path));
        System.out.println(String.format("curator create EPHEMERAL_SEQUENTIAL node :%s successfully.", path1));
    }
```

![image-20220407223438269](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407223438269.png)

**节点的类型通过`create().withMode(CreateMode.xxx)`决定，参数如下图所示：**

![image-20220407222811079](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220407222811079.png)

### 4. 获得节点数据

```java
    // 获得节点数据
    @Test
    public void testGetData() throws Exception {
        byte[] bytes = curatorFramework.getData().forPath("/curator-node");
        System.out.println(new String(bytes));
    }

// 输出：创建持久节点
```

### 5. 修改节点数据

```java
    // 修改节点数据
    @Test
    public void testSetData() throws Exception {
        curatorFramework.setData().forPath("/curator-node", "修改节点数据".getBytes());
        byte[] bytes = curatorFramework.getData().forPath("/curator-node");
        System.out.println(new String(bytes));
    }
// 输出：修改节点数据
```

### 6. 创建子节点的同时创建父节点

```java
    // 创建节点的同时创建父节点
    @Test
    public void testCreateWithParent() throws Exception {
        String path = "/parent/subNode1";
        String s = curatorFramework.create().creatingParentsIfNeeded().forPath(path, "创建节点的同时创建父节点".getBytes());
        System.out.println(String.format("curator create node :%s successfully.", s));
    }
// creatingParentsIfNeeded()表示父节点的不存在的话就创建父节点
```

### 7. 删除子节点的同时删除父节点

```java
    // 删除节点
    @Test
    public void testDelete() throws Exception {
        String path = "/parent";

        curatorFramework.delete().deletingChildrenIfNeeded().forPath(path);
    }
//  curatorFramework.delete().deletingChildrenIfNeeded().forPath(path);表示有子节点就将子节点一起删除
```

## 七、zookeeper实现分布式锁

### 1. zookeeper中锁的类型

- 读锁：⼤家都可以读，要想上读锁的前提：之前的锁没有写锁；
- 写锁：只有得到写锁的才能写。要想上写锁的前提是，之前没有任何锁。

### 2. zookeeper如何上读锁

- 创建⼀个临时序号节点，节点的数据是 `read`，表示是读锁
- 获取当前 `zookeeper` 中序号比自己小的所有节点
- 判断最小节点是否是读锁：
  - 如果不是读锁的话，则上锁失败，为最⼩节点设置监听。阻塞等待，`zookeeper` 的 `watch` 机制会当最⼩节点发⽣变化时通知当前节点，于是再执⾏第⼆步的流程；
  - 如果是读锁的话，则上锁成功。

![image-20220408145413753](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408145413753.png)

### 3. zookeeper 如何上写锁

- 创建⼀个临时序号节点，节点的数据是write，表示是 写锁
- 获取 `zookeeper` 中所有的⼦节点
- 判断⾃⼰是否是最⼩的节点：
  - 如果是，则上写锁成功；
  - 如果不是，说明前⾯还有锁，则上锁失败，监听最⼩的节点，如果最⼩节点有变化，则回到第⼆步。

![image-20220408152351778](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408152351778.png)

### 4. 羊群效应

如果⽤上述的上锁⽅式，只要有节点发⽣变化，就会触发其他节点的监听事件，这样的话对 `zookeeper` 的压⼒⾮常⼤——⽺群效应。

可以调整成 **链式监听** 解决这个问题。

> **即节点值监听之前一个节点的状态。**

![image-20220408152605814](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408152605814.png)

### 5. curator实现读写锁

#### 5.1 获取读锁

```java
    // 获取读锁
    @Test
    public void testGetReadLock() throws Exception {
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(curatorFramework, "/lock1");
        // 获取读锁对象
        InterProcessLock readLock = interProcessReadWriteLock.readLock();
        System.out.println("等待获取读锁对象");
        // 获取锁
        readLock.acquire();
        for (int i = 1; i <= 100; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放锁
        readLock.release();
        System.out.println("释放锁");
    }
```

#### 5.2 获取写锁

```java
    // 获取写锁
    @Test
    public void testGetWriteLock() throws Exception {
        InterProcessReadWriteLock interProcessReadWriteLock =
                new InterProcessReadWriteLock(curatorFramework, "/lock1");
        // 获取读锁对象
        InterProcessLock writeLock = interProcessReadWriteLock.writeLock();
        System.out.println("等待获取写锁对象");
        // 获取锁
        writeLock.acquire();
        for (int i = 1; i <= 100; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放锁
        writeLock.release();
        System.out.println("释放锁");
    }
```

## 八、zookeeper的 watch 机制

### 1. Watch 机制介绍

​		我们可以把 **Watch** 理解成是注册在特定 Znode 上的触发器。当这个 Znode 发⽣改变，也就是调⽤了 `create` ， `delete` ， `setData` ⽅法的时候，将会触发 Znode 上注册的对应事件，请求 **Watch** 的客户端会接收到异步通知。

具体交互过程如下：

- 客户端调⽤ `getData` ⽅法， `watch` 参数是 `true` 。服务端接到请求，返回节点数据，并且在对应的哈希表⾥插⼊被` Watch` 的 `Znode` 路径，以及 `Watcher` 列表。
- 当被 **Watch** 的 `Znode` 已删除，服务端会查找哈希表，找到该 `Znode` 对应的所有 `Watcher`，异步通知客户端，并且删除哈希表中对应的 `Key-Value`。

![image-20220408153943989](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408153943989.png)

客户端使⽤了 NIO 通信模式监听服务端的调⽤

### 2. zkCli客户端使⽤watch

```bash
create /test xxx
get -w /test ⼀次性监听节点
ls -w /test 监听⽬录,创建和删除⼦节点会收到通知。⼦节点中新增节点不会收到通知
ls -R -w /test 对于⼦节点中⼦节点的变化，但内容的变化不会收到通知
```

### 3. curator客户端使⽤watch

```java
    // 监听节点数据变化
    @Test
    public void addNodeLister() throws Exception {
        NodeCache nodeCache = new NodeCache(curatorFramework, "/curator-node");
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println("节点/curator-node 改变");
            }
        });
        nodeCache.start();
        printNodeData();
        System.in.read();
    }

    public void printNodeData() throws Exception {
        byte[] bytes = curatorFramework.getData().forPath("/curator-node");
        System.out.println(new String(bytes));
    }

/*
* 修改节点数据
* 节点/curator-node 改变
* 节点/curator-node 改变
*/
```

## 九、Zookeeper集群实战

### 1.Zookeeper集群角色

`zookeeper` 集群中的节点有三种⻆⾊

- `Leader`：处理集群的所有事务请求(**数据的写、数据的读**)，集群中只有⼀个 `Leader`。
- `Follower`：可以处理**读请求**，也可以**参与 `Leader` 选举**。
- `Observer`：只能处理**读请求**，提升集群读的性能，但**不能参与 `Leader` 选举**。

### 2. 集群搭建

1. 创建4个节点的myid，并设值

   在 `zookeeper` 中创建以下四个⽂件

   ```bash
   /usr/local/zookeeper/zkdata/zk1# echo 1 > myid
   /usr/local/zookeeper/zkdata/zk2# echo 2 > myid
   /usr/local/zookeeper/zkdata/zk3# echo 3 > myid
   /usr/local/zookeeper/zkdata/zk4# echo 4 > myid
   ```

2. 编写4个 `zoo.cfg`

   ```sh
   # The number of milliseconds of each tick
   tickTime=2000
   # The number of ticks that the initial
   # synchronization phase can take
   initLimit=10
   # The number of ticks that can pass between
   # sending a request and getting an acknowledgement
   syncLimit=5
   # 修改对应的zk1 zk2 zk3 zk4
   dataDir=/usr/local/zookeeper/zkdata/zk1
   # 修改对应的端⼝ 2181 2182 2183 2184
   clientPort=2181
   # 2001为集群通信端⼝，3001为集群选举端⼝，observer表示不参与集群选举
   server.1=172.16.253.54:2001:3001
   server.2=172.16.253.54:2002:3002
   server.3=172.16.253.54:2003:3003
   server.4=172.16.253.54:2004:3004:observer
   ```

3. 启动4台 `Zookeeper`

   ```bash
   ./bin/zkServer.sh status ./conf/zoo1.cfg
   ./bin/zkServer.sh status ./conf/zoo2.cfg
   ./bin/zkServer.sh status ./conf/zoo3.cfg
   ./bin/zkServer.sh status ./conf/zoo4.cfg
   ```

4. 连接Zookeeper集群

   ```bash
   ./bin/zkCli.sh -server
   172.16.253.54:2181,172.16.253.54:2182,172.16.253.54:2183
   ```

## 十、ZAB协议

### 1. 什么是ZAB协议

​	`zookeeper` 作为⾮常重要的分布式协调组件，需要进⾏集群部署，集群中会以⼀主多从的形式进⾏部署。

​    `zookeeper`为了保证数据的⼀致性，使⽤了 `ZAB(Zookeeper AtomicBroadcast)`协议，这个协议解决了 `Zookeeper` 的**崩溃恢复和主从数据同步**的问题。

![image-20220408171652910](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408171652910.png)

### 2. ZAB协议定义的四种节点状态

- `Looking` ：选举状态。
- `Following` ：`Follower` 节点（从节点）所处的状态。
- `Leading` ：`Leader` 节点（主节点）所处状态。
- `Observing`：观察者节点所处的状态

### 3. 集群上线时的Leader选举过程

​		`Zookeeper` 集群中的节点在上线时，将会进⼊到 `Looking` 状态，也就是选举 `Leader` 的状态，这个状态具体会发⽣什么？

![image-20220408171840295](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408171840295.png)

### 4. 崩溃恢复时的Leader选举

​		`Leader` 建⽴完后，`Leader` 周期性地不断向 `Follower` 发送⼼跳（ping命令，没有内容的 `socket` ）。当 `Leader` 崩溃后，`Follower` 发现 `socket` 通道已关闭，于是 `Follower` 开始进⼊到 `Looking` 状态，重新进行 `Leader` 选举，此时集群不能对外提供服务。

### 5. 主从服务器之间的数据同步

![image-20220408173733726](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408173733726.png)

### 6. Zookeeper中的NIO与BIO的应用

- `NIO`
  - ⽤于被客户端连接的 `2181` 端⼝，使⽤的是 `NIO` 模式与客户端建⽴连接；
  - 客户端开启 `Watch` 时，也使⽤ `NIO` ，等待 `Zookeeper` 服务器的回调。
- `BIO`
  - 集群在选举时，多个节点之间的投票通信端⼝，使⽤ `BIO` 进⾏通信。

## 十一、CAP理论

### 1. CAP 定理

​		2000 年 7 ⽉，加州⼤学伯克利分校的 Eric Brewer 教授在 ACM PODC 会议上提出 CAP 猜想。2年后，麻省理⼯学院的 Seth Gilbert 和 Nancy Lynch 从理论上证明了 CAP。之后，CAP 理论正式成为分布式计算领域的公认定理。

CAP 理论为：⼀个分布式系统最多只能同时满足 `⼀致性(Consistency)`、`可⽤性(Availability)`和 `分区容错性(Partition tolerance)`这三项中的两项。

- **`⼀致性(Consistency)`**

  ⼀致性指 “all nodes see the same data at the same time”，即更新操作成功并返回客户端完成后，所有节点在同⼀时间的数据完全⼀致。

- **`可⽤性(Availability)`**

  可⽤性指“Reads and writes always succeed”，即服务⼀直可⽤，⽽且是正常响应时间。

- **`分区容错性(Partition tolerance)`**

  分区容错性指“the system continues to operate despite arbitrary message loss or failureof part of the system”，即分布式系统在遇到某节点或⽹络分区故障的时候，仍然能够对外提供满⾜⼀致性或可⽤性的服务。即 避免单点故障，就要进⾏冗余部署，冗余部署相当于是服务的分区，这样的分区就具备了容错性。

### 2. CAP 权衡

​		对于涉及到钱财这样不能有⼀丝让步的场景，C 必须保证。⽹络发⽣故障宁可停⽌服务，这是保证 CA，舍弃 P。貌似这⼏年国内银⾏业发⽣了不下 10 起事故，但影响⾯不⼤，报到也不多，⼴⼤群众知道的少。还有⼀种是保证 CP，舍弃 A。例如⽹络故障是只读不写。

![image-20220408194634573](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220408194634573.png)

### 3. BASE 理论

​			eBay 的架构师 Dan Pritchett 源于对⼤规模分布式系统的实践总结，在 ACM 上发表⽂章提出BASE 理论，BASE 理论是对 CAP 理论的延伸，核⼼思想是**即使⽆法做到强⼀致性（StrongConsistency，CAP 的⼀致性就是强⼀致性），但应⽤可以采⽤适合的⽅式达到最终⼀致性（Eventual Consitency）。**

- **基本可⽤（Basically Available）**
  - 基本可⽤是指分布式系统在出现故障的时候，允许损失部分可⽤性，即保证核⼼可⽤。
  - 电商⼤促时，为了应对访问量激增，部分⽤户可能会被引导到降级⻚⾯，服务层也可能只提供降级服务。这就是损失部分可⽤性的体现。
- **软状态（Soft State）**
  - 软状态是指允许系统存在中间状态，⽽该中间状态不会影响系统整体可⽤性。分布式存储中⼀般⼀份数据⾄少会有三个副本，允许不同节点间副本同步的延时就是软状态的体现。mysql replication 的异步复制也是⼀种体现。
- **最终⼀致性（Eventual Consistency）**
  - 最终⼀致性是指系统中的所有数据副本经过⼀定时间后，最终能够达到⼀致的状态。弱⼀致性和强⼀致性相反，最终⼀致性是弱⼀致性的⼀种特殊情况。

### 4. Zookeeper追求的⼀致性

`Zookeeper` 在数据同步时，追求的并不是强⼀致性，⽽是**顺序⼀致性（事务id的单调递增）**。