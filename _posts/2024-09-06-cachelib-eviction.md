---
layout:     post
title:      "【CacheLib】Eviction Policy"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
---

# LRU
CacheLib 采用的是 `带可选配置` 的 LRU算法，以下是涉及的特定术语
- `Tail age` : is the time the item in the LRU tail spent in the cache. If Tins(tail) is the time when an item was inserted into the cache, tail_age = now() - Tins(tail). 
- `Insertion point (IP)` : is a position in LRU where a new item is inserted. 
  - Classic LRU IP is the head of the list. Moving an item from its current position in LRU to the head is called promotion.

![1](/img/cachelib/lru.png)

主要的修改包含以下两点
1. `custom IP`: which is not at the head of the list. New items will be inserted at the position ipos(p) = size(LRU) * p from the tail, where p ∈ [0,1] is a ratio parameter.  For example, suppose the regular LRU tail age be 600s, but you want to cache only items that were accessed during the 200s after insertion. Inserting new items at position ipos(1/3) from the tail may help you to achieve this. Note that custom IP affects only items that're inserted into the cache the first time; it doesn't affect items that on access are moved to the head of the list.
    - `custom IP`: 默认为0，即在LRU head位置插入新item，但是可以通过ratio parameter，控制插入的position.

2. `promotion delay`: 将访问命中的item移到head的延迟. Normally every access item is moved to the LRU head. In fact, you need to be precise about promotions from the tail of LRU and don't bother(无须费心) promoting items close to the head. Before promotion, we check the time since the last promotion and promote only if it is longer than the delay parameter `T`. This is `performance optimization`. In the setup of the last example, you may want to promote only items with tail age of 300s, roughly when they get to the bottom half of the LRU.
    - 核心的点：对于靠近list head的item，无需对该item进行promotion，优化访问的性能

### Configuration
- `lruRefreshTime`: How often does cachelib refresh a previously accessed item. By default this is 60 seconds.
  - 先前访问过的item多久刷新一次，默认值为60s

- `updateOnWrite/updateOnRead` Specifies if a LRU promotion happens on read or write or both. 
  - As a rule of thumb, for most services that care primarily about read performance, turn on updateOnRead. //大多数服务只关心读性能，所以只需要打开updateOnRead即可
  - However, if your service cares a lot about retention time of items that are recently written//如果服务非常关注写入item的保留时间, then turn on updateOnWrite as well.
  - 参数的默认值：updateOnRead = true and updateOnWrite = false.

- `ipSpec` This essentially turns the LRU into a two-segmented LRU(参数将LRU分成两段LRU). Setting this to 1 means every new insertion will be inserted 1/2 from the end of the LRU, 2 means 1/4 from the end of the LRU, and so on.
  - value = 0，即在LRU head位置插入新item
  - value = 1, 插入位置为LRU tail的1/2位置
  - value = 2, 插入位置为LRU tail的1/4位置
