# Redis 개관

- 출처: https://try.redis.io/

**Redis** is a 
  - **key-value store** that stores some data (value) inside a key.
  - **data structure server** becuase each value can contain a complex data structure (string/list/hashes/sorted sets/hyperloglog).
  - All Redis operations implemented by single comands are *atomic*.


## Basics

- SET/GET/EXISTS

```
> SET server:name "fido"
OK
> GET server:name
"fido"
> EXISTS server:name
(integer) 1
> EXISTS server:blabla
(integer) 0
```

- DEL/INCR
  - INCR: automatically increment a number stored at a given key.
  - INCR for String: you are implementing **counters**.
  - INCR is an *atomic* operation.

```
> SET connections 10
OK
> INCR connections
(integer) 11
> INCR connections
(integer) 12
> DEL connections
(integer) 1
> INCR connections
(integer) 1
> INCRBY connections 100
(integer) 101
> DECRBY connections 10
(integer) 91
```

- EXPIRE/TTL/SET/PERSIST
  - EXPIRE: tell a key only exist for certain length of time.
    - return 0: fail
    - return 1: success
  - TTL: check how long a key will exist.
    - return -2: no such key
    - return -1: no expire setting (never expire)
  - PEXPIRE/PTTL: use milliseconds instead of seconds.
  - SET: TTL will be reset.
  - PERSIST: remove expire and make key permanent

```
> SET resource:lock "Redis Demo"
OK
> EXPIRE resouce:lock 120
(integer) 0
> TTL resouce:lock
(integer) -2
> TTL resource:lock
(integer) -1
> EXPIRE resource:lock 120
(integer) 1
> TTL resource:lock
(integer) 113
> TTL resource:lock
(integer) 111
```

```
> SET resource:lock "Redis Demo 3" EX 5
OK
> TTL resource:lock
(integer) 3
```

```
> SET resource:lock "Redis Demo 3" EX 5
OK
> PERSIST resource:lock
(integer) 1
> TTL resource:lock
(integer) -1
```

## List

- A list is a series of ordered values.
- Fast to access elements near top/bottom.
- RPUSH/LPUSH/LLEN/LRANGE/LPOP/RPOP
  - RPUSH/LPUSH: directly create key and add new elements.
    - return: number of elements
    - can specify multiple elements
  - LPOP/RPOP: automatically remove keys that will result empty.
  - LRANGE: gives subset of the list

```
> RPUSH friends "Alice"
(integer) 1
> RPUSH friends "Bob"
(integer) 2
> LPUSH friends "Sam"
(integer) 3
```

```
> LRANGE friends 0 -1
1) "Sam"
2) "Alice"
3) "Bob"
> LRANGE friends 0 1
1) "Sam"
2) "Alice"
> LRANGE friends 1 2
1) "Alice"
2) "Bob"
> LRANGE friends 0 -2
1) "Sam"
2) "Alice"
```

```
> LPOP friends
"Sam"
> RPOP friends
"Bob"
> LLEN friends
(integer) 1
> LRANGE friends 0 -1
1) "Alice"
```

```
> RPUSH friends 1 2 3
(integer) 4
> LLEN friends
(integer) 4
```

## Set

- Similar to list, doesn't have order, each element may only appear once.
- Very fast to test for membership.
- SADD/SREM/SISMEMBER/SMEMBERS/SUNION
  - SADD: variadic
    - return 1: value is not there
    - return 0: value is already there
  - SREM
    - return 1: value was there
    - return 0: value was not there
  - SISMEMBER
    - return 1: value is there
    - return 0: value is not there
  - SMEMBERS: list of all members
  - SUNION: combines two or more sets and returns list of all elements.

```
> SADD superpowers "flight"
(integer) 1
> SADD superpowers "x-ray vision" "reflexes"
(integer) 2
> SREM superpowers "reflexes"
1
> SREM superpowers "making pizza"
0
```

```
> SISMEMBER superpowers "flight"
(integer) 1
> SISMEMBER superpowers "reflexes"
(integer) 0
> SMEMBERS superpowers
1) "flight"
2) "x-ray vision"
> SADD birdpowers "pecking"
(integer) 1
> SADD birdpowers "flight"
(integer) 1
> SUNION superpowers birdpowers
1) "flight"
2) "x-ray vision"
3) "pecking"
```

- SPOP: extracts 2 elements in random

```
> SADD letters a b c d e f
(integer) 6
> SPOP letters 2
1) "a"
2) "f"
```

## Sorted Sets

- Similar to regular sets, each value has an associated score used to sort the elemenets in set.

```
> ZADD hackers 1940 "Alan Kay"
(integer) 1
> ZADD hackers 1906 "Grace Hopper"
(integer) 1
> ZADD hackers 1953 "Richard Stallman"
(integer) 1
> ZADD hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> ZADD hackers 1916 "Claude Shannon"
(integer) 1
> ZADD hackers 1969 "Linus Torvalds"
(integer) 1
> ZADD hackers 1957 "Sophie Wilson"
(integer) 1
> ZADD hackers 1912 "Alan Turing"
(integer) 1
> ZRANGE hackers 2 4
1) "Claude Shannon"
2) "Alan Kay"
3) "Richard Stallman"
```

## Hashes

- Hashes are maps between string feilds and string values.
- Perfect data type to represent objects.

- HSET/HGETALL/HMSET/HGET

```
> HSET user:1000 name "John Smith"
1
> HSET user:1000 email "john.smith@example.com"
1
> HSET user:1000 password "s3cret"
1
> HGETALL user:1000
1) "name"
2) "John Smith"
3) "email"
4) "john.smith@example.com"
5) "password"
6) "s3cret"
> HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"
OK
> HGET user:1001 name
"Mary Jones"
```

- Numerical values in hash fields are handled exactly the same as simple strings.
- HSET/HINCRBY/HDEL

```
> HSET user:1000 name "John Smith"
1
> HSET user:1000 email "john.smith@example.com"
1
> HSET user:1000 password "s3cret"
1
> HGETALL user:1000
1) "name"
2) "John Smith"
3) "email"
4) "john.smith@example.com"
5) "password"
6) "s3cret"
> HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"
OK
> HGET user:1001 name
"Mary Jones"
```
