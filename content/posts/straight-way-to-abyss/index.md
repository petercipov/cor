---
title: "Straight way to abyss: Premature optimization"
date: 2013-10-31
description: "Our servers hit 95% CPU while traffic was normal. The cause was a hidden cache inside a JSON library, a classic case of premature optimization."
featureAlt: "Red chalk study of a twisting human figure."
summary: ""
tags: ["premature optimisation", "anti patterns", "assumptions"]
---

It happened some time ago. Some of our servers went crazy. Traffic was normal, but out of the blue CPU was at 95%. It seemed to be a tough nut of a day. I looked for the culprit in stack traces with my colleagues and found this place in the code:

```java
public JSONObject createJSON(String raw) {
	return new JSONObject(raw);
}
```

Creating a simple JSON object caused the infinite loop?! WOW. JSONObject uses internally java.util.HashMap. After some googling, there was a single culprit candidate - race condition.

But how is it possible? There is no shared state. Object is created on the heap and it is not shared between threads. My mind was torn into two pieces. One was 100% sure that this is a race condition but the second one told me the opposite.

Out of sheer frustration we started to look at the source code of the JSON library. And we found it!

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

I was stunned for a while. A static map shared between all instances of JSONObject - the mind-blower of the day. This solved all our questions. This was the wanted critical section. We could not reveal it before because it happened only sometimes when HashMap internal threshold was reached (we did not use more than 100 JSON unique keys before :DD ). This bug was fortunately fixed in the library after half a year.

This is my personal encounter with the anti-pattern "Premature optimization". It caused a whole day of searching, patching source codes and redeploying server instances. A lot for a simple sunny day...