---
title: 创建Zookeeper连接的同步异步方法记录
date: 2018-10-28 23:18:40
categories:
	- Java
tags:
	- Zookeeper
	- 分布式协调
---
摘要：Zookeeper实践
<!-- more -->

## 建立Zookeeper回话机制
Zookeeper的API都是围绕着句柄(handle)而构建的，在每次API调用的时候也都是传递的这个句柄，他代表了和Zookeeper的一次回话，

### 获取Zookeeper管理权限的两种方式
同步方式
```java
public class Master implements Watcher{
    ZooKeeper zk;
    String hostPort;
    String serverId = Long.toString(new Random().nextLong()); // 唯一标志节点的随机值
    static boolean isLeader = false; 默认不是主节点
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
                isLeader = new String(data).equals(serverId);
                return true;
            } catch (KeeperException | InterruptedException e) {
                // 没有master节点 return fasle
                return false;
            }

        }
    }
    // 同步的 runForMaster方法

//    void runForMaster() {
//        while (true) {
//            try {
//                zk.create("/master", serverId.getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
//                //
//                isLeader = true;
//                break;
//            } catch (KeeperException e) {
//
//            } catch (InterruptedException e) {
//
//            }
//            // 如果存在主节点，就退出
//            if (checkMaster()) {
//                break;
//            }
//        }
//
//    }

```
这里的同步代码我给注释掉了，因为在大多数ZK应用里面，为了保证并发情况下的低延迟，可以再单线程中进行多个调用，所以采用异步方式。
我们需要创建异步调用对象
```java
static AsyncCallback.StringCallback masterCreateCallBack = new AsyncCallback.StringCallback() {
    @Override
    public void processResult(int rc, String path, Object ctx, String name) {
    
    }
}
```
processResult各参数的含义：  
rc: 返回调用的结构，返回OK或者异常对应的Code类中的编码  
path: 传给create的path 参数值
ctx： 传给create的上下文参数  
name ： 创建的znode节点名称  
在processResult重载函数中判断连接状态
```java
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
```

完成以后，就可以在create的异步调用函数中就可以使用这个masterCreateCallBack了。  
```java
// 异步的 runForMaster方法
void runForMaster() {

    zk.create("/master", serverId.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL, masterCreateCallBack, null);


}
```
这里的`ZooDefs.Ids.OPEN_ACL_UNSAFE`,代表ZK创建节点的权限，我第一次写得时候竟然自己把ACL包装成了List结构传进去，结果报了` txntype:-1 reqpath:n/a Error Path:/master Error:KeeperErrorCode = InvalidACL for /master`异常。ZK已经为我们写好了权限的常量。
```java
public interface Ids {
    Id ANYONE_ID_UNSAFE = new Id("world", "anyone");
    Id AUTH_IDS = new Id("auth", "");
    ArrayList<ACL> OPEN_ACL_UNSAFE = new ArrayList(Collections.singletonList(new ACL(31, ANYONE_ID_UNSAFE)));
    ArrayList<ACL> CREATOR_ALL_ACL = new ArrayList(Collections.singletonList(new ACL(31, AUTH_IDS)));
    ArrayList<ACL> READ_ACL_UNSAFE = new ArrayList(Collections.singletonList(new ACL(1, ANYONE_ID_UNSAFE)));
}
```
异步调用的返回码，也是规定好的。
```java
public interface OpCode {
    int notification = 0;
    int create = 1;
    int delete = 2;
    int exists = 3;
    int getData = 4;
    int setData = 5;
    int getACL = 6;
    int setACL = 7;
    int getChildren = 8;
    int sync = 9;
    int ping = 11;
    int getChildren2 = 12;
    int check = 13;
    int multi = 14;
    int auth = 100;
    int setWatches = 101;
    int sasl = 102;
    int createSession = -10;
    int closeSession = -11;
    int error = -1;
}
用KeeperException.Code.get(rc)获取即可。
```






