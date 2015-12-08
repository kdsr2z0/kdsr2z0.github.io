---
layout: post
title:  "Object Convert"
date:   2015-06-28 15:28:00
categories: java
tags: shit
shortUrl: 
---


import org.codehaus.jackson.map.ObjectMapper;


Object To Map
---------------

Map map = new ObjectMapper().convertValue(myObject, Map.class);


Map to Object
---------------
MyClass myObject = new ObjectMapper().convertValue(map, MyClass.class);

