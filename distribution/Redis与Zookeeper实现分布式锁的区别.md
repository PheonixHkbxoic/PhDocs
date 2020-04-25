# Redis实现分布式锁

　　1.根据lockKey区进行setnx（set not exist，如果key值为空，则正常设置，返回1，否则不会进行设置并返回0）操作，如果设置成功，表示已经获得锁，否则并没有获取锁。

　　2.如果没有获得锁，去Redis上拿到该key对应的值，在该key上我们存储一个时间戳（用毫秒表示，t1），为了避免死锁以及其他客户端占用该锁超过一定时间（5秒），使用该客户端当前时间戳，与存储的时间戳作比较。

　　3.如果没有超过该key的使用时限，返回false，表示其他人正在占用该key，不能强制使用；如果已经超过时限，那我们就可以进行解锁，使用我们的时间戳来代替该字段的值。

　　4.但是如果在setnx失败后，get该值却无法拿到该字段时，说明操作之前该锁已经被释放，这个时候，最好的办法就是重新执行一遍setnx方法来获取其值以获得该锁。

　　释放锁：删除redis中key



```java
 public class RedisKeyLock {
     private static Logger logger = Logger.getLogger(RedisKeyLock.class);
     private final static long ACCQUIRE_LOCK_TIMEOUT_IN_MS = 10 * 1000;
     private final static int EXPIRE_IN_SECOND = 5;//锁失效时间
     private final static long WAIT_INTERVAL_IN_MS = 100;
     private static RedisKeyLock lock;
     private JedisPool jedisPool;
     private RedisKeyLock(JedisPool pool){
         this.jedisPool = pool;
     }
     public static RedisKeyLock getInstance(JedisPool pool){
         if(lock == null){
             lock = new RedisKeyLock(pool);
         }
         return lock;
     }
     public void lock(final String redisKey) {
         Jedis resource = null;
         try {
             long now = System.currentTimeMillis();
             resource = jedisPool.getResource();
             long timeoutAt = now + ACCQUIRE_LOCK_TIMEOUT_IN_MS;
             boolean flag = false;
             while (true) {
                 String expireAt = String.valueOf(now + EXPIRE_IN_SECOND * 1000);
                 long ret = resource.setnx(redisKey, expireAt);
                 if (ret == 1) {//已获取锁
                     flag = true;
                     break;
                 } else {//未获取锁，重试获取锁
                     String oldExpireAt = resource.get(redisKey);
                     if (oldExpireAt != null && Long.parseLong(oldExpireAt) < now) {
                         oldExpireAt = resource.getSet(redisKey, expireAt);
                         if (Long.parseLong(oldExpireAt) < now) {
                             flag = true;
                             break;
                         }
                     }
                 }
                 if (timeoutAt < now) {
                     break;
                 }
               TimeUnit.NANOSECONDS.sleep(WAIT_INTERVAL_IN_MS);
             }
             if (!flag) {
                 throw new RuntimeException("canot acquire lock now ...");
             }
         } catch (JedisException je) {
             logger.error("lock", je);
             je.printStackTrace();
             if (resource != null) {
                 jedisPool.returnBrokenResource(resource);
             }
         } catch (Exception e) {
             e.printStackTrace();
             logger.error("lock", e);
         } finally {
             if (resource != null) {
                 jedisPool.returnResource(resource);
             }
         }
     }
     public boolean unlock(final String redisKey) {
         Jedis resource = null;
         try {
             resource = jedisPool.getResource();
             resource.del(redisKey);
             return true;
         } catch (JedisException je) {
             je.printStackTrace();
             if (resource != null) {
                 jedisPool.returnBrokenResource(resource);
             }
             return false;
         } catch (Exception e) {
             logger.error("lock", e);
             return false;
         } finally {
             if (resource != null) {
                 jedisPool.returnResource(resource);
             }
         }
     }
 }
```



另一个版本：

　　SET my:lock 随机值 NX PX 30000

　　这个的NX的意思就是只有key不存在的时候才会设置成功，PX 30000的意思是30秒后锁自动释放。别人创建的时候如果发现已经有了就不能加锁了。

　　释放锁就是删除key，但是一般可以用lua脚本删除，判断value一样才删除

　　为啥要用随机值呢？因为如果某个客户端获取到了锁，但是阻塞了很长时间才执行完，此时可能已经自动释放锁了，此时可能别的客户端已经获取到了这个锁，要是你这个时候直接删除key的话会有问题，所以得用随机值加上面的lua脚本来释放锁。（就是根据这个随机值来判断这个锁是不是自己加的）

　　如果是Redis是单机，会有问题。因为如果是普通的redis单实例，那就是单点故障。单节点挂了会导致锁失效。

　　如果是redis普通主从，那redis主从异步复制，如果主节点挂了，key还没同步到从节点，此时从节点切换为主节点，别人就会拿到锁。

 

RedLock算法

　　这个场景是假设有一个redis cluster，有5个redis master实例。然后执行如下步骤获取一把锁：

　　获取当前时间戳，单位是毫秒

　　跟上面类似，轮流尝试在每个master节点上创建锁，过期时间较短，一般就几十毫秒

　　尝试在大多数节点上建立一个锁，比如5个节点就要求是3个节点（n / 2 +1）

　　客户端计算建立好锁的时间，如果建立锁的时间小于超时时间，就算建立成功了

　　要是锁建立失败了，那么就依次删除这个锁

　　只要别人建立了一把分布式锁，你就得不断轮询去尝试获取锁

 

![img](https://img2018.cnblogs.com/blog/720994/201812/720994-20181209150457361-385299448.png)

 

　

# Zookeeper实现分布式锁

基于临时顺序节点：

　　1.客户端调用create()方法创建名为“locknode/guid-lock-”的节点，需要注意的是，这里节点的创建类型需要设置为EPHEMERAL_SEQUENTIAL。

　　2.客户端调用getChildren(“locknode”)方法来获取所有已经创建的子节点。

　　3.客户端获取到所有子节点path之后，如果发现自己在步骤1中创建的节点是所有节点中序号最小的，那么就认为这个客户端获得了锁。

　　4.如果创建的节点不是所有节点中序号最小的，那么则监视比自己创建节点的序列号小的最大的节点，进入等待。直到下次监视的子节点变更的时候，再进行子节点的获取，判断是否获取锁。

　　释放锁的过程相对比较简单，就是删除自己创建的那个子节点即可。

 

不太严谨的代码：

```java
   public class ZooKeeperDistributedLock implements Watcher{
     private ZooKeeper zk;
     private String locksRoot= "/locks";
     private String productId;
     private String waitNode;
     private String lockNode;
     private CountDownLatch latch;
     private CountDownLatch connectedLatch = new CountDownLatch(1);
 private int sessionTimeout = 30000; 
     public ZooKeeperDistributedLock(String productId){
         this.productId = productId;
          try {
        String address = "192.168.31.187:2181,192.168.31.19:2181,192.168.31.227:2181";
             zk = new ZooKeeper(address, sessionTimeout, this);
             connectedLatch.await();
         } catch (IOException e) {
             throw new LockException(e);
         } catch (KeeperException e) {
             throw new LockException(e);
         } catch (InterruptedException e) {
             throw new LockException(e);
         }
     }
     public void process(WatchedEvent event) {
         if(event.getState()==KeeperState.SyncConnected){
             connectedLatch.countDown();
             return;
         }
         if(this.latch != null) {  
             this.latch.countDown(); 
         }
     }
     public void acquireDistributedLock() {   
         try {
             if(this.tryLock()){
                 return;
             }
             else{
                 waitForLock(waitNode, sessionTimeout);
             }
         } catch (KeeperException e) {
             throw new LockException(e);
         } catch (InterruptedException e) {
             throw new LockException(e);
         } 
 }
     public boolean tryLock() {
         try {
          // 传入进去的locksRoot + “/” + productId
         // 假设productId代表了一个商品id，比如说1
         // locksRoot = locks
         // /locks/10000000000，/locks/10000000001，/locks/10000000002
             lockNode = zk.create(locksRoot + "/" + productId, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
             // 看看刚创建的节点是不是最小的节点
          // locks：10000000000，10000000001，10000000002
             List<String> locks = zk.getChildren(locksRoot, false);
             Collections.sort(locks);
             if(lockNode.equals(locksRoot+"/"+ locks.get(0))){
                 //如果是最小的节点,则表示取得锁
                 return true;
             }
             //如果不是最小的节点，找到比自己小1的节点
       int previousLockIndex = -1;
             for(int i = 0; i < locks.size(); i++) {
         if(lockNode.equals(locksRoot + “/” + locks.get(i))) {
                      previousLockIndex = i - 1;
             break;
         }
        }
        this.waitNode = locks.get(previousLockIndex);
         } catch (KeeperException e) {
             throw new LockException(e);
         } catch (InterruptedException e) {
             throw new LockException(e);
         }
         return false;
     }
     private boolean waitForLock(String waitNode, long waitTime) throws InterruptedException, KeeperException {
         Stat stat = zk.exists(locksRoot + "/" + waitNode, true);
         if(stat != null){
             this.latch = new CountDownLatch(1);
             this.latch.await(waitTime, TimeUnit.MILLISECONDS);                   this.latch = null;
         }
         return true;
 }
     public void unlock() {
         try {
         // 删除/locks/10000000000节点
         // 删除/locks/10000000001节点
             System.out.println("unlock " + lockNode);
             zk.delete(lockNode,-1);
             lockNode = null;
             zk.close();
         } catch (InterruptedException e) {
             e.printStackTrace();
         } catch (KeeperException e) {
             e.printStackTrace();
         }
 }
     public class LockException extends RuntimeException {
         private static final long serialVersionUID = 1L;
         public LockException(String e){
             super(e);
         }
         public LockException(Exception e){
             super(e);
         }
 }
 // 如果有一把锁，被多个人给竞争，此时多个人会排队，第一个拿到锁的人会执行，然后释放锁，后面的每个人都会去监听排在自己前面的那个人创建的node上，一旦某个人释放了锁，排在自己后面的人就会被zookeeper给通知，一旦被通知了之后，就ok了，自己就获取到了锁，就可以执行代码了
 }  
```



另一个版本：

　　zk分布式锁，就是某个节点尝试创建临时znode，此时创建成功了就获取了这个锁；这个时候别的客户端来创建锁会失败，只能注册个监听器监听这个锁。

　　释放锁就是删除这个znode，一旦释放掉就会通知客户端，然后有一个等待着的客户端就可以再次重新加锁。





 　　redis分布式锁，其实需要自己不断去尝试获取锁，比较消耗性能

　　zk分布式锁，获取不到锁，注册个监听器即可，不需要不断主动尝试获取锁，性能开销较小

　　另外一点就是，如果是redis获取锁的那个客户端bug了或者挂了，那么只能等待超时时间之后才能释放锁；而zk的话，因为创建的是临时znode，只要客户端挂了，znode就没了，此时就自动释放锁





## 链接

https://www.cnblogs.com/mengchunchen/p/9647756.html