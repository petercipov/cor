---
title: "Straight way to abyss: Premature optimization"
date: 2013-10-31
description: "Our servers hit 95% CPU while traffic was normal. The cause was a hidden cache inside a JSON library, a classic case of premature optimization."
featureAlt: "Red chalk study of a twisting human figure."
summary: ""
tags: ["premature optimisation", "anti patterns", "assumptions"]
---

It happened some time ago. Some of our servers went crazy: traffic was normal, but CPU usage suddenly jumped to 95%. It looked like it would be a tough day. My colleagues and I searched the stack traces for the culprit and found this piece of code:

```java
public JSONObject createJSON(String raw) {
	return new JSONObject(raw);
}
```

Creating a simple JSON object caused an infinite loop?! WOW. JSONObject uses java.util.HashMap internally, and after some googling we had only one suspect: a race condition.

But how was that possible? There was no shared state. The object was created on the heap and never shared between threads. My mind was torn in two: one half was 100% sure this was a race condition, while the other insisted it could not be.

Out of sheer frustration, we started reading the source code of the JSON library. And we found it!

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

I was stunned for a while. A static map shared between all instances of JSONObject: that was the mind-blower of the day. It answered all our questions, because here was the critical section we had been looking for. We had not seen the problem before because it appeared only when the HashMap reached its internal threshold. Until then, we had never used more than 100 unique JSON keys :DD. Fortunately, the bug was fixed in the library half a year later.

This was my personal encounter with the anti-pattern of premature optimization. It cost us a whole day of searching, patching source code and redeploying servers. Quite a lot for a simple sunny day...
