# BigCache

A big, fast cache.

BigCache stores keys on JVM heap memory, value data on Offheap memory first, when configured Offheap memory size is used up, more value data will be persisted to disk file.
Basically there is no limit how much value data can be stored as long as you have enough disk space, this is why we name it BigCache, however, since BigCache still use JVM heap memory to store keys, in practice, BigCache is suitable for scenario where the number of keys is less than 20 million while value data are much larger than keys.

 
# Feature Highlight:
1. **Big**: when configured offheap memory is used up, more data will be persisted to disk, no data eviction issue like Memcached.
2. **High Read/Write Performance**: average read/write performance close to O(1) memory access, worst read/write performance close to O(1) disk access, tailored for caching/session data scenarios.
3. **Thread Safe**: supporting multi-threads concurrent access.
4. **Expiration & Compaction**: automation expired and deleted data cleanup, automatic free space compaction, avoiding memory and disk space waste.
5. **Light in Desigin & Implementation**: simple Map like interface, only support Get/Put/Delete operations, cross platform Java based, small codebase size, embeddable.

# Performance Highlight:
Suppose 10 bytes key and 100 bytes value, on average PC, random read/write can be **> 500,000** ops/sec.

## The Design
![BigCache Design](https://raw.githubusercontent.com/ctriposs/bigcache/master/doc/bigcache.png)
#### Design Essentials
1. Keys are stored on JVM heap memory(ConcurrentHashMap).
2. Values are stored on fix sized block, block can be pure file block, memory mapped file block or offheap block, see ***Configuration section*** for storage Mode configuration. 
3. There are two background threads, ***CleanerThread*** periodically cleans expired Key/Values, ***MoverThread*** periodically moves Key/Values out of blocks in low usage rate(because of expiration or deletion), then returns the freed block to Free Block Pool.
4. For optimization, update operation will reuse original space if possible, and stripped write/read lock is leveraged for better concurrency.


## How to Use

## Configuration
You can configure BigCache via [CacheConfig](https://raw.githubusercontent.com/ctriposs/bigcache/master/src/main/java/com/ctriposs/bigcache/CacheConfig.java) object.

#### 1. storageMode 
Three storage modes are supported:
> **Pure file**, all value data is persisted on disk file, this is the ***default***.    
> **Memory Mapped file + Pure file**, inital data is stored on memory mapped file, when configured ***maxOffHeapMemorySize*** is used up, more data will be persisted on disk file.    
> **Offheap Memory + Pure file**, inital data is stored on offheap memory, when configured maxOffHeapMemorySize is usded up, more data will be persisted to disk file.       

#### 2. maxOffHeapMemorySize 
How much offheap memory size will be used for inital value data storage, when configured offheap memory is used up, more data will be persisted to disk. No effect when the storageMode is ***Pure file***. Unit:bytes, ***default is 2 * 1024 * 1024 * 1024 = 2GB***.

##Copyright and License
Copyright 2014 ctriposs

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this work except in compliance with the License. You may obtain a copy of the License in the LICENSE file, or at:

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

