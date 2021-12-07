---
title: 'dispatch_semaphore_t 信号量'
date: '2018-11-28'
spoiler: ''
---

  # `dispatch_semaphore_create`

`dispatch_semaphore_create(long value)` 传入的是一个数字,该数字必须大于等于0。列如传入10，表示创建一个信号总量为10信号量（就好比一共有10个停车位）。假如传入0

The starting value for the semaphore. Passing a value less than zero will cause NULL to be returned.

则返回为NULL


# `dispatch_semaphore_wait`

该函数会使传入的信号量减1,当信号量有 >= 1时,就会继续往下执行。如果没有,该线程就会一直处于等待状态，可阻塞当前线程。该函数需要传入另一个参数 

`#define DISPATCH_TIME_NOW  表示从现在开始 #define DISPATCH_TIME_FOREVER  表示永不超时`


# `dispatch_semaphore_single`

该函数会使传入的信号量加1,与`dispatch_ semaphore _ wait`成对出现。

拿YYKit源码来举例：在#import "UIButton+YYWebImage.m"中


```obj-c
- (instancetype)init {
    self = [super init];
    //声明一个信号总量为1信号量
    _lock = dispatch_semaphore_create(1);
    _dic = [NSMutableDictionary new];
    return self;
}

- (_YYWebImageSetter *)setterForState:(NSNumber *)state {

//是当前线程处于阻塞状态,直到有 >= 1 信号量才会继续执行下面的取值操作
    dispatch_semaphore_wait(_lock, DISPATCH_TIME_FOREVER);
    //如果不加入线程锁，就很有可能发生多个线程同时访问_dic字典,造成资源竞争。
    _YYWebImageSetter *setter = _dic[state];
    //使该信号量加1。保持信号量的平衡，如果没有保持信号量的平衡,可能会出现莫名的bug,甚至crash。
    dispatch_semaphore_signal(_lock);
    return setter;
}


- (_YYWebImageSetter *)lazySetterForState:(NSNumber *)state {

    //同理
    dispatch_semaphore_wait(_lock, DISPATCH_TIME_FOREVER);
    _YYWebImageSetter *setter = _dic[state];
    if (!setter) {
        setter = [_YYWebImageSetter new];
        _dic[state] = setter;
    }
    dispatch_semaphore_signal(_lock);
    return setter;
}
```

在读取_YYWebImageSetter该函数中使用`dispatch_semaphore_wait`和`dispatch_semaphore_signal`使当前函数为线程安全。实际上信号量就是锁，它与 `@synchronized` 函数看起来很像。所起到的作用就是至始至终只会有一个线程去访问该函数。

在 [这边文章中](http://www.cnblogs.com/snailHL/p/3906112.html)很形象的比喻了信号量

```
关于信号量，一般可以用停车来比喻。

　　停车场剩余4个车位，那么即使同时来了四辆车也能停的下。如果此时来了五辆车，那么就有一辆需要等待。

　　信号量的值就相当于剩余车位的数目，dispatch_semaphore_wait函数就相当于来了一辆车，dispatch_semaphore_signal

　　就相当于走了一辆车。停车位的剩余数目在初始化的时候就已经指明了（dispatch_semaphore_create（long value）），

　　调用一次dispatch_semaphore_signal，剩余的车位就增加一个；调用一次dispatch_semaphore_wait剩余车位就减少一个；

　　当剩余车位为0时，再来车（即调用dispatch_semaphore_wait）就只能等待。有可能同时有几辆车等待一个停车位。有些车主

　　没有耐心，给自己设定了一段等待时间，这段时间内等不到停车位就走了，如果等到了就开进去停车。而有些车主就像把车停在这，

　　所以就一直等下去。
　　```
举个🌰   NSMutableArray不是线程安全的,我们可以稍作修饰,线程安全的去更新数组

　```obj-c
 	NSMutableArray *array = [NSMutableArray array];
   dispatch_semaphore_t semphore = dispatch_semaphore_create(1);
   dispatch_queue_t concurrentQueue = dispatch_queue_create("com.lx.gcddemo",DISPATCH_QUEUE_CONCURRENT);
        dispatch_apply(100, concurrentQueue, ^(size_t i) {
            dispatch_semaphore_wait(semphore, DISPATCH_TIME_FOREVER);
            /*
            @synchronized (array) {
                [array addObject:[NSNumber numberWithUnsignedInteger:i]];
            }
             */
            [array addObject:[NSNumber numberWithUnsignedInteger:i]];
            dispatch_semaphore_signal(semphore);
        });
        NSLog(@"%@",array);
```
  