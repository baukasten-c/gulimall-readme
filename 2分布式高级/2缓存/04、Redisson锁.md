# Redisson锁

1、lock锁

```java
@ResponseBody
@GetMapping(value = "/lock")
public void lock() {
    //获取锁。只要锁的名字一样，就是同一把锁
    RLock myLock = redisson.getLock("my-lock");
    //加锁
    myLock.lock(20, TimeUnit.SECONDS);
    try {
        System.out.println("加锁成功，执行业务..." + Thread.currentThread().getId());
        try { 
            TimeUnit.SECONDS.sleep(20); 
        } catch (InterruptedException e) { 
            e.printStackTrace(); 
        }
    } catch (Exception ex) {
        ex.printStackTrace();
    } finally {
        //解锁
        System.out.println("释放锁..." + Thread.currentThread().getId());
        myLock.unlock();
    }
}
```

1-1、myLock.lock() 加锁，默认锁的超时时间为30秒，其余线程阻塞式等待

1-2、只要占锁成功，就会启动一个定时任务，重新给锁设置过期时间，定时任务每隔10秒都会自动续期，续成30秒(看门狗默认时间)。业务时间超长，锁也不会因为过期而被释放

1-3、业务只要运行完成，就不会给当前锁续期，即使不手动解锁，锁默认会在30秒内自动过期，不会产生死锁问题

1-4、myLock.lock(10, TimeUnit.SECONDS) 加锁，自定义锁的超时时间，锁不会自动续期

2、读写锁

```java
@GetMapping(value = "/write")
@ResponseBody
public String writeValue() {
    String s = "";
    RReadWriteLock readWriteLock = redisson.getReadWriteLock("rw-lock");
    //改数据加写锁
    RLock rLock = readWriteLock.writeLock();
    try {
        rLock.lock();
        s = UUID.randomUUID().toString();
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        ops.set("writeValue", s);
        TimeUnit.SECONDS.sleep(10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        rLock.unlock();
    }
    return s;
}
@GetMapping(value = "/read")
@ResponseBody
public String readValue() {
    RReadWriteLock readWriteLock = redisson.getReadWriteLock("rw-lock");
    //读数据加读锁
    RLock rLock = readWriteLock.readLock();
    try {
        rLock.lock();
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        s = ops.get("writeValue");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        rLock.unlock();
    }
    return s;
}
```

2-1、读写锁保证一定能获取最新数据，修改期间，写锁是一个排它锁(互斥锁、独享锁)，读锁是一个共享锁

2-2、读 + 读 ：相当于无锁，并发读，会同时加锁成功，会在Redis中记录当前所有的读锁

2-3、写 + 读 ：必须等待写锁释放

2-4、写 + 写 ：阻塞方式

2-5、读 + 写 ：有读锁，写也需要等待。只要读写同时存在，都必须等待

3、信号量

```java
//模拟停车场
@GetMapping(value = "/park")
@ResponseBody
public String park() throws InterruptedException {
    RSemaphore park = redisson.getSemaphore("park");
    //占一个车位，即获取一个信号(redis中信号量值-1)
    park.acquire();   
    return "ok";
}

@GetMapping(value = "/go")
@ResponseBody
public String go() {
    RSemaphore park = redisson.getSemaphore("park");
    //释放一个车位，即释放一个信号(redis中信号量值+1)
    park.release();     
    return "ok";
}
```

3-1、park.acquire()，如果redis中信号量为0，阻塞，直到信号量大于0，可以拿到信号量才会继续执行

3-2、park.tryAcquire()，非阻塞，信号量大于0返回true，信号量为0返回false。可以用于分布式限流

```java
boolean flag = park.tryAcquire();
if (flag) {
    //执行业务
} else {
    //进行限流
}
```

4、闭锁

```java
//模拟放学锁门
@GetMapping(value = "/lockDoor")
@ResponseBody
public String lockDoor() throws InterruptedException {
    RCountDownLatch door = redisson.getCountDownLatch("door");
    //设置等待数
    door.trySetCount(5);
    //等待闭锁完成
    door.await();       
    return "可以锁门了...";
}
@GetMapping(value = "/gogogo/{id}")
@ResponseBody
public String gogogo(@PathVariable("id") Long id) {
    RCountDownLatch door = redisson.getCountDownLatch("door");
    door.countDown();       
    return id + "班的人都走了...";
}
```

5、Redisson锁底层都使用Lua 脚本，可以保证锁的原子性