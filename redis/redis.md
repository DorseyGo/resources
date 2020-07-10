# Redis

## 1. Introduction

Redis is an open-source, <font color="#aa0">in-memory</font> data structure store, used as <font color="#aa0">database</font>, <font color="#aa0">cache</font> and <font color="#aa0">message broker</font>.

## 2. Data Structure

Redis is not a <i>plain</i> key-value store, it is actually a <i>data structure server</i>, supporting different kinds of values. In Redis, the value is <b>NOT</b> limited to a simple string, but can also hold more complex data structures.

- <u>Binary-safe strings</u>
- <u>Lists</u>, collections of string elements sorted according to the order of insertion. they are basically <font color="#aa0">linked list</font>.
- <u>Sets</u>, collections of unique, unsorted string elements.
- <u>Sorted Sets</u>, similar to Sets but where every string element is associated to a floating number value, called <font color="#aa0">score</font>. The elements are always taken sorted by their score, so unlike Sets it is possible to retrieve a <i>range</i> of elements.
-  <u>Hashes</u>, which are maps composed of fields associated with values. Both the field and value are strings.
- <u>Bit arrays</u>, it is possible, using special commands, to handle String values like an array of bits: you can set and clear individual bits, count all the bits set to 1, find the first set or unset bit, and so forth.
- <u>HyperLogLogs</u>, this is a probabilistic data structure which is used in order to estimate the cardinality of a set.
- <u>Streams</u>, append-only collections of map-like entries that provide an abstraction log data type. 

### 2.1 Strings

The simplest type of values you can associate with a Redis key. The string data type is useful for a number of use cases, such as <font color="#aa0">caching HTML fragments or pages</font>.

#### 2.1.1 Commands

Commands that played on strings,

- GET() - retrieve a string value
- SET() - replace the existing value already stored into the key, in the case that the key already exists. It also has some interesting options, for example to fail it if the key already exists (<font color="#aa0">NX</font>), or the opposite (<font color="#aa0">XX</font>).
- INCR() - parses the string value as an integer, increments it by one, and finally sets the obtained value as the new value. same to <font color="#aa0">DECR, INCRBY, DECRBY</font>.
- GETSET() - sets a key to a new value, returning the old value as the result
- MSET() & MGET() - ability to set or retrieve the value of multiple keys in a single command
- EXISTS() - returns 1 or 0 to signal if a given key exists or not
- DEL() - deletes the key and associated value, no matter the value
-  TYPE() - returns the type of the value for the specified key

##### 2.1.1.1 Redis Expire

Basically you set a timeout for a key, which is limited time to live. when the time elapses, the key is automatically destroyed, exactly as if the user called DEL command with the key.

- can be set both using seconds or milliseconds precision
- the expire time resolution is always 1 millisecond
- information about expires are replicated and persisted on disk, the time virtually passes when your Redis server remains stopped

>EXPIRE() - to set the timeout or via SET() command with extra option <font color="#aa0">EX</font>

### 2.2 Redis Lists

Redis lists are implemented via Linked Lists. This means that even if you have millions of elements inside a list, the operation of adding a new element in the head or in the tail of the list is performed in <font color="#aa0"><i>constant</i></font> time.

#### 2.2.1 Commands

The <font color="#aa0">LPUSH</font> command adds a new element into a list, on the left (at the head), while <font color="#aa0">RPUSH</font> command adds a new element into a list, on the right (at the tail).

The <font color="#aa0">LRANGE</font> command extracts ranges of elements from lists.

#### 2.2.2 Common use cases for lists

Two very representative use cases are the following,

- Remember the latest updates posted by users into a social network
- communication between processes, using a <font color="#aa0">consumer-producer</font> pattern where the producer pushes items into a list, and a consumer (usually a <font color="#aa0">worker</font>) consumes those items and executed actions.