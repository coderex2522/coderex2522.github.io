---
layout:     post
title:      "【CacheLib实现】Cache find flow"
subtitle:   ""
author:     "rex"
header-img: "img/present/1.jpg"
header-mask:  0.5
catalog: true
tags:

    - CacheLib
    - 代码学习
---

# find
```cpp
typename CacheAllocator<CacheTrait>::ReadHandle
CacheAllocator<CacheTrait>::find(typename Item::Key key) {
    return findImpl(key, AccessMode::kRead);
}
```

### findImpl
```cpp

```



# findInternal
```cpp
WriteHandle findInternal(Key key) {
    // Note: this method can not be const because we need  a non-const
    // reference to create the ItemReleaser.
    return accessContainer_->find(key);
}
```

# findInternalWithExpiration
```cpp
typename CacheAllocator<CacheTrait>::WriteHandle
CacheAllocator<CacheTrait>::findInternalWithExpiration(
    Key key, AllocatorApiEvent event) {

auto handle = findInternal(key);
if (!handle) {
    return;
}

if (UNLIKELY(handle->isExpired())) {
    WriteHandle ret;
    ret.markExpired();
    return ret;
}

return handle;
}
```

