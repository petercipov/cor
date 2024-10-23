---
title: "Straight way to abyss: Premature optimization"
date: 2013-10-31
description: ""
summary: ""
tags: ["premature optimisation", "anti patterns", "assumptions"]
---

It happened some time ago. Some of our servers went crazy. Traffic was in normal, but out of the blank CPU was 95%. It seemed to be a nutshell of a day. I looked for the culprit in stack traces with my colleagues and found this place in code

```java
public JSONObject createJSON(String raw) {
	return new JSONObject(raw);
}
```

Creating simple JSON object caused the infinite loop?! WOW. JSONObject uses internally java.util.HashMap. After some googling, there was a single culprit candidate - race condition.

But how it is posssible? There is no shared state. Object is created on the heap and it is not shared between threads. My mind was torn to two pieces. One was 100% sure that this is a race condition but the second one told me the oposite.

Out of cheer frustration we started to look to the sourcecodes of JSON library. And we found it !

```java
public class JSONObject {
    /**
     * The maximum number of keys in the key pool.
     */
     private static final int size = 100;

   /**
     * Key pooling is like string interning, but without
     * permanently tying up memory. To help conserve
     * memory, storage of duplicated key strings in
     * JSONObjects will be avoided by using a key pool
     * to manage unique key string objects. This is
     * used by JSONObject.put(string, object).
     */
     private static HashMap keyPool = new HashMap(size);
     
     public JSONObject(String source) throws JSONException {
     	...
        //cache parsed keys
        String pooled = (String)keyPool.get(key);
        if (pooled == null) {        	
            keyPool.put(key, key);
        }
        ...
    }
 }
```

I stayed stunned for a while. A static map shared between all instances of JSONObject - the mind-blower of a day. This solved all our questions. This was the wanted critical section. We could not reveal it before because it happened only sometimes when HashMap internal threshold was reached (we did not use more than 100 JSON unique keys before :DD ). This bug was hopefully fixed in the library after half a year.

This is my personal encounter with anti-pattern "Premature optimization". It caused whole day of searching, patching source codes and redeploying server instances. A lot for a simple sunny day...