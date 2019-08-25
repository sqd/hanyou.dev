---
title: "Amortized LRU Cache (Now WITHOUT Linked List!)"
date: 2019-06-03 17:51:00
tags: "Data Structure"
---
As you probably know LRU cache is a common interview question. Just for the record, I know that the canonical way of doing this is using a hashmap with a linked list, but to implement a generic linked list correctly is somewhat unpleasant when there are special cases such as deleting from the head.

Anyway, I came up with this LRU cache implementation that also works in constant time without having to use a linked list. The code should be pretty straight-forward with the comments.

```python
CONST_FACTOR = 2

class LRUCache:
    def __init__(self, max_size):
        self.max_size = max_size
        # A list of the keys we keep in cache
        # The larger the index, the more recent a key is
        self.keys = list()
        # A dict from key to value
        self.lookup = dict()

    def assign(self, key, value):
        self.lookup[key] = value
        # key become the most recently used
        self.renew(key)
    
    def query(self, key):
        if key in self.lookup:
            value = self.lookup[key]
            # key becomes the most recently used
            self.renew(key)
            return value

    def renew(self, key):
        # Make this key the most recently used
        self.keys.append(key)
        # Try to prune
        self.prune()

    def prune(self):
        # Postpone pruning
        if len(self.keys) < CONST_FACTOR*self.max_size:
            return

        old_lookup, self.lookup = self.lookup, dict()
        old_keys, self.keys = self.keys, list()
        # Going from the most recent to the least recent by iterating recersely
        for key in reversed(old_keys):
            # If under size limit
            if len(self.keys) < self.max_size:
                # Only use the most recent value for this key
                if key not in self.lookup:
                    self.lookup[key] = old_lookup[key]
                    self.renew(key)
            # If reach size limit, discard the rest
            else:
                break
```

So why is this good? Suppose we insert keys $1, 2 ... nk-1, nk$ to this LRU with a constant factor $k$ and maximum size $n$. For $1, 2 ...$ all the way through $nk-1$, clearly the operation is constant. For $nk$, however, we have to do a prune. How bad is that? In this specific case, we have to loop through $n$ elements, but let's take the theoretical worst case and say we have to loop through $O(nk)$ elements. But wait! Amortized onto $nk$ insertions, $O(nk)$ become just $O(1)$.

This wouldn't actually pass that LeetCode problem though. Because it is able to keep more values in memory then the set maximum number.