---
layout: post
comments: true
title: Creating An In-Memory FIFO Cache
author: malike
categories: [tech]
tags: [documentation,sample]
image:
    path: /posts/linkedhashmap-queues.jpg
    width: 800
    height: 500
alt: .
redirect_from: "/Queue-LinkedHashMap/"
---


 A well implemented HashMap in Java would give you performance of *O(1)* for _get_ and _put_.
Well implemented in this case means your _hashCode_ and _equals_ are helpful to reduce collisions.

From Java 7 to Java 8 the HashMap implementation changed from using  *linkedList* to *balance trees* to resolve collisions, this means for very bad implementation (sometimes there are just collisions) of _hashCode_ which may result in many collisions we'll no longer have a LinkedList of items,which would give us *O(n)* worst case. Balance trees improve this with a *O(log n)* from Java 8.
This [article](https://dzone.com/articles/hashmap-performance) talks about it in depth.



Now to the main point of this article, there are many In-Memory Key-Pair databases there. Eg: Redis. But just in case you want one built with HashMap data type. Which can get you performance of O(1) for both ***put*** and ***get*** without using a framework HashMaps would be our go-to data structure. 

But the problem with using HashMap as a cache is,if you don't control the size it might blow up your memory. 
The default HashMap can grow in size. 
If memory is not your problem you are good to go with HashMap but if you want to control the size and the items to be stored with FIFO priorities then let's continue.

Unfortunately HashMaps don't maintain the order of items when inserted. *Why do we need the order?*, we need the order to determine what gets to *leave* the queue. That is know the order the items were entered.

LinkedHashMap however maintains the order and also has [removeEldestEntry](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html#removeEldestEntry-java.util.Map.Entry-)

By simply extending the properties of [removeEldestEntry](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html#removeEldestEntry-java.util.Map.Entry-). We can build an in-memory cache with fixed size HashMap with O(1) performance for ***get*** and ***push*** operations.

```java
	public class HashMapCache<K,V> extends LinkedHashMap{

		private final int maxSize;

	    public FixedSizeHashMap(int maxSize) {
	        this.maxSize = maxSize;
	    }

	    @Override
	    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
	        return size() > maxSize;
	    }
	    
	    @Override
	    public boolean equals(Object obj){
	        return obj==this;
	    }
	    
	    @Override
	    public int hashCode(){
	        return System.identityHashCode(this);
	    }

	}
```


Lets write a test to test our implementation. You can see I've not written tests for all the behaviors of a ListHashMap. 
Our main focus is to see if your ***canPushEarliestOut()***  would be <span style="color:#1DA31D">***green***</span>.

```java
	public class HashMapCacheTest{

	@Before
	    public void setup() {

	        fixedSizeHashMap.put("one", "one");
	        fixedSizeHashMap.put("two", "two");
	        fixedSizeHashMap.put("three", "three");

	    }

	    @Test
	    public void hasRightSize() {
	        Assert.assertEquals(3, fixedSizeHashMap.size());
	        fixedSizeHashMap = new FixedSizeHashMap<>(5);
	        Assert.assertEquals(0, fixedSizeHashMap.size());
	    }

	    @Test
	    public void canAdd() {
	        Assert.assertTrue(fixedSizeHashMap.containsKey("one"));
	        Assert.assertEquals(3, fixedSizeHashMap.size());
	    }

	    @Test
	    public void canGet() {
	        Assert.assertTrue(fixedSizeHashMap.containsKey("one"));
	        Assert.assertTrue(fixedSizeHashMap.containsKey("two"));
	        Assert.assertTrue(fixedSizeHashMap.containsKey("three"));
	        Assert.assertEquals(3, fixedSizeHashMap.size());
	    }

	    @Test
	    public void canPushEarliestOut() {
	        Assert.assertTrue(fixedSizeHashMap.containsKey("one"));
	        Assert.assertEquals(3, fixedSizeHashMap.size());
	        fixedSizeHashMap.put("four", "four");
	        Assert.assertFalse(fixedSizeHashMap.containsKey("one"));
	        Assert.assertTrue(fixedSizeHashMap.containsKey("four"));
	        Assert.assertEquals(3, fixedSizeHashMap.size());
	    }


	}
```



<br>
<br>
**REFERENCES**

[https://dzone.com/articles/hashmap-performance](https://dzone.com/articles/hashmap-performance)  -  Tomasz Nurkiewicz

