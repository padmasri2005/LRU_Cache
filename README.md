# LRU_Cache
## Caching and Its Implementation

### 1. What is Caching? Need for Caching
Caching is a technique used to store frequently accessed data in a temporary storage layer to reduce access latency and improve performance. It minimizes repeated computations or expensive data retrieval operations by providing a faster, in-memory alternative to fetching data from a slower primary storage system. Caching is crucial in various computing applications, including databases, web applications, and operating systems, to enhance efficiency and scalability.

### 2. In-Memory Cache: Redis and Memcached Introduction
- **Redis**: Redis (Remote Dictionary Server) is an open-source, in-memory key-value store that supports various data structures such as strings, hashes, lists, sets, and sorted sets. It is widely used for caching, real-time analytics, and pub/sub messaging.
- **Memcached**: Memcached is a high-performance, distributed memory caching system that stores data in memory to reduce database load and increase application responsiveness. It follows a simple key-value storage model and is widely used in web applications.

### 3. Cache Memory in Computer Organization
Cache memory is a small, high-speed memory located close to the CPU that stores frequently accessed instructions and data. It bridges the speed gap between the main memory (RAM) and the processor. Cache memory operates in multiple levels:
- **L1 Cache**: Integrated into the processor, fastest but smallest in size.
- **L2 Cache**: Larger than L1, slightly slower but still faster than RAM.
- **L3 Cache**: Shared across multiple processor cores, improves performance in multi-core systems.

### 4. Different Cache Replacement Strategies
When a cache is full, it must decide which data to replace. Common cache replacement policies include:
- **Least Recently Used (LRU)**: Evicts the least recently accessed data.
- **Least Frequently Used (LFU)**: Removes the least frequently accessed items.
- **First-In-First-Out (FIFO)**: Discards the oldest cache entry.
- **Random Replacement (RR)**: Replaces a randomly selected entry.

### 5. Designing LRU Cache Using Doubly Linked List and HashMap
An LRU cache can be efficiently implemented using a combination of a doubly linked list and a hashmap:
- **HashMap (O(1) Lookup)**: Maps keys to nodes in the doubly linked list.
- **Doubly Linked List (O(1) Insertion & Deletion)**: Maintains the order of access, with the most recently used elements at the front and the least recently used at the back.

### 6. UML Diagram for LRU Cache Design
```
+---------------+
|   LRUCache    |
|---------------|
| - capacity    |
| - size        |
| - mp (Map)    |
| - ll (CDLL)   |
|---------------|
| + get(k)      |
| + put(k, v)   |
+---------------+
        |
        v
+---------------+
|     CDLL      |
|---------------|
| - head        |
|---------------|
| + insAtBegin  |
| + delLast     |
| + moveToFront |
+---------------+
        |
        v
+----------------+
|    CDLLNode    |
|----------------|
| - key, val     |
| - prev, next   |
+----------------+
```

### 7. Code Implementation with Explanation
The following Java implementation of an LRU cache uses a circular doubly linked list (CDLL) for efficient insertion, deletion, and reordering:

```java
import java.util.*;

class LRUCache {
    CDLL ll;
    int capacity;
    int size;
    Map<Integer, CDLLNode> mp;
    
    public LRUCache(int cap){
        ll = new CDLL();
        this.capacity = cap;
        this.size = 0;
        mp = new HashMap<>();
    }
    
    int get(int k){
        if(mp.containsKey(k)){
            CDLLNode node = mp.get(k);
            ll.moveToFront(node); // Move accessed node to front
            return node.val;
        }
        return -1; // Key not found
    }
    
    void put(int k, int v){
        if(mp.containsKey(k)){ // Update existing key
            CDLLNode node = mp.get(k);
            node.val = v;
            ll.moveToFront(node);
        } else { // Insert new key
            if(size < capacity){
                CDLLNode nn = ll.insAtBegin(k, v);
                mp.put(k, nn);
                size++;
            } else { // Evict LRU
                int delKey = ll.delLast();
                mp.remove(delKey);
                CDLLNode nn = ll.insAtBegin(k, v);
                mp.put(k, nn);
            }
        }
    }
}

class CDLLNode {
    int key, val;
    CDLLNode prev, next;
    
    public CDLLNode(int k, int v){
        this.key = k;
        this.val = v;
        this.prev = this.next = null;
    }
}

class CDLL {
    CDLLNode head;
    
    public CDLL(){
        head = null;
    }
    
    CDLLNode insAtBegin(int k, int v){
        CDLLNode nn = new CDLLNode(k, v);
        nn.next = nn; nn.prev = nn;
        if(head == null) {head = nn; return nn;}
        CDLLNode last = head.prev;
        nn.next = head; head.prev = nn;
        last.next = nn; nn.prev = last;
        head = nn; return nn;
    }
    
    int delLast(){
        if(head == null) return -1;
        CDLLNode last = head.prev;
        int ret = last.key;
        if(last == head) { head = null; return ret;}
        last.prev.next = head;
        head.prev = last.prev;
        return ret;
    }
    
    void moveToFront(CDLLNode nodeToMove){
        if(nodeToMove == head) return; // Already at front
        nodeToMove.prev.next = nodeToMove.next;
        nodeToMove.next.prev = nodeToMove.prev;
        CDLLNode last = head.prev;
        nodeToMove.next = head; head.prev = nodeToMove;
        last.next = nodeToMove; nodeToMove.prev = last;
        head = nodeToMove;
    }
}

public class Main {
    public static void main(String[] args) {
        LRUCache cache = new LRUCache(3);
        cache.put(1, 10);
        cache.put(2, 20);
        cache.put(3, 30);
        System.out.println(cache.get(1)); // Should print 10
        cache.put(4, 40);
        System.out.println(cache.get(2)); // Should print -1 (evicted)
    }
}
```
