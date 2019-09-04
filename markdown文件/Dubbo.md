1）**select**方法作用是从**invokers**选出下一个被调用的**invoker**

2） Random LoadBalance  、RoundRobinLoadBalance。。。



源码：**AbstractLoadBalance**

![img](D:\OneDrive - std.uestc.edu.cn\markdown文件\151439_EONF_113011.jpg)

各节点的关系：参考：[this][https://my.oschina.net/ywbrj042/blog/689818?p=1]

- 这里的Invoker是Provider的**一个可调用Service的抽象**，**Invoker封装了Provider地址及Service接口信息。**
- Directory代表多个Invoker，可以把它看成List<Invoker>，但与List不同的是，它的值可能是动态变化的，比如注册中心推送变更。（暂不用看）
- Cluster将Directory中的多个Invoker伪装成一个Invoker，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个。（暂不用看）
- Router负责从多个Invoker中按路由规则选出子集，比如读写分离，应用隔离等。（暂不用看）
- LoadBalance负责从多个Invoker中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，**调用失败后，需要重选**。

继承AbstractLoadBalance这个模板类，在这个模板类中封装了getWeight方法，**获取invoker的权重**，特别的是，这个权重和预热时间有关，只有提供者在线时长到达了预热时间，调用这个方法获取invoker权重的时候，才能获得100%的权重。在子类中获取invoker权重都是调用这个方法.

> 个人猜测：getWeight方法，获取invoker的权重实际上就是获取provider的权重。
>
> Invoker与Invocation的关系类似Class与Class实例化。

```java
//计算预热权重
    protected int getWeight(Invoker<?> invoker, Invocation invocation) {
        int weight = invoker.getUrl().getMethodParameter(invocation.getMethodName(), Constants.WEIGHT_KEY, Constants.DEFAULT_WEIGHT);
        if (weight > 0) {
            long timestamp = invoker.getUrl().getParameter(Constants.REMOTE_TIMESTAMP_KEY, 0L);
            if (timestamp > 0L) {
                //提供者在线时长
                int uptime = (int) (System.currentTimeMillis() - timestamp);
                //预热时间默认10分钟
                int warmup = invoker.getUrl().getParameter(Constants.WARMUP_KEY, Constants.DEFAULT_WARMUP);
                if (uptime > 0 && uptime < warmup) {
                    weight = calculateWarmupWeight(uptime, warmup, weight);
                }
            }
        }
        return weight;
    }

//用于计算预热权重
    static int calculateWarmupWeight(int uptime, int warmup, int weight) {
        int ww = (int) ((float) uptime / ((float) warmup / (float) weight));
        return ww < 1 ? 1 : (ww > weight ? weight : ww);
    }
```

**AbstractLoadBalance**:  实现了select方法，增加了对invoker数量的判断，如果只有一个直接返回，**invokers超过1个才需要使用负载均衡选择逻辑**，具体负载均衡逻辑由子类实现doSelect方法

```
@Override
    public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        if (invokers == null || invokers.isEmpty())
            return null;
        //如果只有一个提供者直接返回，预热失效
        if (invokers.size() == 1)
            return invokers.get(0);
        return doSelect(invokers, url, invocation);
    }

    //让子类实现doSelect
    protected abstract <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation);
```

> 为什么要预热，jvm运行时会对字节码进行优化，刚启动的字节码肯定不是最优的。或者是提供者本身有其他缓存需要初始化之类的。所以预热是有必要的。不要一启动就和其他提供者承受同样流量，可能效率会变慢。当然，如果只有一个提供者的情况下，预热就失效了。

### RandomLoadBalance

随机算法，如果每个invokers权重一样，那么就是普通的随机算法，如果不同就是加权随机

```java
protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
    
        int length = invokers.size(); // Number of invokers
        int totalWeight = 0; // The sum of weights
        boolean sameWeight = true; // Every invoker has the same weight?
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);//第一个weight
            totalWeight += weight; // Sum
            if (sameWeight && i > 0//至少有两个invoker
                    && weight != getWeight(invokers.get(i - 1), invocation)) {
                sameWeight = false;
            }
        }
        //如果提供者权重不一样，加权随机    based on totalWeight.
        if (totalWeight > 0 && !sameWeight) {
            // If (not every invoker has the same weight & at least one invoker's weight>0), select randomly based on totalWeight.
            int offset = random.nextInt(totalWeight);
            // Return a invoker based on the random value.
            for (int i = 0; i < length; i++) {
                offset -= getWeight(invokers.get(i), invocation);
                if (offset < 0) {
                    return invokers.get(i);
                }
            }
        }
        //如果提供者权重都一样，普通随机
        // If all invokers have the same weight value or totalWeight=0, return evenly.
        return invokers.get(random.nextInt(length));
    }
```

### RoundRobinLoadBalance

轮训算法。如果每个invoker权重一样，就是普通的轮训算法。如果不同，是加权的轮训算法。

```java
protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        String key = invokers.get(0).getUrl().getServiceKey() + "." + invocation.getMethodName();
        int length = invokers.size(); // Number of invokers
        int maxWeight = 0; // The maximum weight
        int minWeight = Integer.MAX_VALUE; // The minimum weight
        final LinkedHashMap<Invoker<T>, IntegerWrapper> invokerToWeightMap = new LinkedHashMap<Invoker<T>, IntegerWrapper>();
        int weightSum = 0;
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);////获取权重值
            maxWeight = Math.max(maxWeight, weight); // Choose the maximum weight
            minWeight = Math.min(minWeight, weight); // Choose the minimum weight
            if (weight > 0) {
                invokerToWeightMap.put(invokers.get(i), new IntegerWrapper(weight));
                weightSum += weight;
            }
        }
        AtomicPositiveInteger sequence = sequences.get(key);
        if (sequence == null) {
            sequences.putIfAbsent(key, new AtomicPositiveInteger());
            sequence = sequences.get(key);
        }
        int currentSequence = sequence.getAndIncrement();
        //如果每个提供者权重不一样，采用加权轮训
        if (maxWeight > 0 && minWeight < maxWeight) {
            int mod = currentSequence % weightSum;
            for (int i = 0; i < maxWeight; i++) {
                for (Map.Entry<Invoker<T>, IntegerWrapper> each : invokerToWeightMap.entrySet()) {
                    final Invoker<T> k = each.getKey();
                    final IntegerWrapper v = each.getValue();
                    if (mod == 0 && v.getValue() > 0) {
                        return k;
                    }
                    if (v.getValue() > 0) {
                        v.decrement();
                        mod--;
                    }
                }
            }
        }
        //每个服务提供者权重一样，就是普通轮训
        // Round robin
        return invokers.get(currentSequence % length);
    }
```

