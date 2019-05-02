---
title: Zookeeper实现主从模式
date: 2018-11-12 23:18:40
categories:
	- Zookeeper
tags:
	- Zookeeper
---
ZK 探究
<!-- more -->

## native Zookeeper主从模式
## Zk的master节点
```java
public class Master implements Watcher{
    ZooKeeper zk;
    String hostPort;
    String serverId = Long.toString(new Random().nextLong()); // 唯一标志节点的随机值
    boolean isLeader = false; // 自己是不是主节点? 默认不是主节点
    Master(String hostPort) {
        this.hostPort = hostPort;
    }
    void startZK() throws Exception{
        zk = new ZooKeeper(hostPort, 5000, this);
    }

    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }
    void stopZK () throws Exception{
        zk.close();
    }
    // 返回是否存在一个master节点
    boolean checkMaster() {
        while (true) {
            try {
                Stat stat = new Stat();
                byte[] data = zk.getData("/master", false, stat);
                System.out.println(new String(data).toString());
                System.out.println(serverId);
                isLeader = new String(data).equals(serverId);
                return true;
            } catch (KeeperException | InterruptedException e) {
                // 没有master节点 return fasle
                return false;
            }

        }
    }

    // 回调函数
     AsyncCallback.StringCallback masterCreateCallBack = new AsyncCallback.StringCallback() {
        @Override
        public void processResult(int rc, String path, Object ctx, String name) {
            switch (KeeperException.Code.get(rc)) {
                // 连接丢失，但我们还要确定master节点是否被创建了，checkMaster
                case CONNECTIONLOSS:
                    checkMaster();
                    return;
                case OK:
                    isLeader = true;
                    break;
                default:
                    isLeader = false;
            }
            System.out.println(KeeperException.Code.get(rc));
            System.out.println("我" + (isLeader ? "是" : "不是") + "the leader");
        }
    };

// 异步的 runForMaster方法
    void runForMaster() {

        zk.create("/master", serverId.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL, masterCreateCallBack, null);


    }
    // 使用异步API来设置元数据
    public void bootstrap() {
        // 没有数据传入这些znode节点，传入空的字节数组
        createParent("/workers", new byte[0]);
        createParent("/assign", new byte[0]);
        createParent("/tasks", new byte[0]);
        createParent("/status", new byte[0]);
    }

    void createParent(String path, byte[] data) {
        zk.create(path, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT, createParentCallBack,data);
    }

    AsyncCallback.StringCallback createParentCallBack = new AsyncCallback.StringCallback() {
        @Override
        public void processResult(int rc, String path, Object ctx, String name) {
            switch (KeeperException.Code.get(rc)) {
                case CONNECTIONLOSS:
                    createParent(path, (byte[]) ctx);
                    break;
                case OK:
                    System.out.println("Parent created");

                    // 节点已存在
                case NODEEXISTS:
                    System.out.println("Parent already registered " + path);

                default:
                    System.out.println("错误" + KeeperException.create(KeeperException.Code.get(rc)));

            }
        }
    };

    public static void main(String[] args) throws Exception{
        Master master = new Master("127.0.0.1:2181");
        master.startZK();
        master.runForMaster();
        Thread.sleep(3000);
        if (master.isLeader) {
            System.out.println("自己成为了主节点");
            Thread.sleep(4000);

        } else {
            System.out.println("其他人成为了主节点");
        }
        // 依次创建 /tasks /assign /worker 目录
//        master.bootstrap();
        Thread.sleep(600000);
        master.stopZK();

    }
}
```
## 注册从节点事件
```java
public class Worker implements Watcher {
    private static final Logger LOG = LoggerFactory.getLogger(Worker.class);
    ZooKeeper zk;
    String hostport;
    String status;
    String serverID = Integer.toHexString(new Random().nextInt());

    public Worker(String hostport) {
        this.hostport = hostport;
    }
    void startZK() throws IOException {
        zk = new ZooKeeper(hostport, 15000, this);
    }


    @Override
    public void process(WatchedEvent watchedEvent) {
        LOG.info(watchedEvent.toString() + ", " + hostport);
    }
    void register() {
        zk.create("/workers/worker-" + serverID,
                "Idle".getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL,
                createWorkerCallBack, null);
    }
    AsyncCallback.StringCallback createWorkerCallBack = new AsyncCallback.StringCallback() {
        @Override
        public void processResult(int rc, String path, Object ctx, String name) {
            switch (KeeperException.Code.get(rc)) {
                case CONNECTIONLOSS:
                    register(); // 连接丢失，重试
                    break;
                case OK:
                    LOG.info("Registered  successfully: " + serverID);
                    break;
                    // 节点已存在
                case NODEEXISTS:
                    LOG.warn("Already registered:" + serverID);
                    break;
                default:
                    LOG.error("出错:" + KeeperException.create(KeeperException.Code.get(rc)));

            }
        }
    };

    // 加锁的方法
    synchronized private void updateStatus(String status) {
        if (status == this.status) {
            zk.setData("/workers/" + serverID, status.getBytes(), -1, statusUpdateCallBack, status);

        }
    }
    public void setStatus(String status) {
        this.status = status;
        updateStatus(status);
    }

    AsyncCallback.StatCallback statusUpdateCallBack = new AsyncCallback.StatCallback() {
        @Override
        public void processResult(int rc, String path, Object ctx, Stat stat) {
            switch (KeeperException.Code.get(rc)) {
                case CONNECTIONLOSS:

                    break;
                case OK:
                    System.out.println("Parent created");

                    // 节点已存在
                case NODEEXISTS:
                    System.out.println("Parent already registered " + path);

                default:
                    System.out.println("错误" + KeeperException.create(KeeperException.Code.get(rc)));

            }
        }
    };
    public static void main(String[] args) throws Exception{
        Worker worker = new Worker("127.0.0.1:2181");
        worker.startZK();

        worker.register();

        TimeUnit.SECONDS.sleep(600); // 休眠三十秒
    }
}
```
## 任务队列话处理
```java
public class Client implements Watcher {
    ZooKeeper zk;
    String hostport;

    public Client(String hostprot) {
        this.hostport = hostprot;
    }
    void startZK() throws Exception{
        zk = new ZooKeeper(hostport, 15000, this);
    }

    String queueCommand(String command) throws Exception {
        String name = "";
        while (true) {
            try {
                name = zk.create("/tasks/task-",command.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
                return name;
            } catch (KeeperException.NodeExistsException e) {
                throw  new Exception(name + "快要被运行了");
            } catch (KeeperException.ConnectionLossException e) {

            }
        }
    }
     @Override
    public void process(WatchedEvent watchedEvent) {
         System.out.println(watchedEvent);
    }

    public static void main(String[] args) throws Exception{
        Client client = new Client("127.0.0.1:2181");

        client.startZK();

        String name = client.queueCommand("My command 1");
        System.out.println("Created" + name);
    }
}

```
## 再搭配一个管理者，美滋滋。。
```java
public class AdminClient implements Watcher {
    ZooKeeper zk;
    String hostport;

    public AdminClient(String hostport) {
        this.hostport = hostport;
    }
    void  startZK() throws Exception{
        zk = new ZooKeeper(hostport, 15000, this);
    }
    void listState() throws KeeperException, InterruptedException {
        try {
            Stat stat = new Stat();
            byte[] masterData = zk.getData("/master", false, stat);
            Date startDate = new Date(stat.getCtime());
            System.out.println("Master" + new String(masterData) + " since " + startDate);

        } catch (KeeperException.NoNodeException e) {
            System.out.println("No master znode");
        }
        System.out.println("Workers:");
        for (String w : zk.getChildren("/workers", false)) {
            byte[] data = zk.getData("/workers/" + w, false, null);
            String state = new String(data);
            System.out.println("\n" + w + ": " + state);
        }

        System.out.println("Tasks:");
        for (String t: zk.getChildren("/assign", false)) {
            System.out.println("\n" + t);
        }

    }
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }

    public static void main(String[] args) throws Exception{
        AdminClient adminClient = new AdminClient("127.0.0.1:2181");
        adminClient.startZK();
        adminClient.listState();
    }
}
```