---
layout: post
title: "JSON in Android"
subtitle: ""
date: 2018-01-15
author: Aether
category: coding
tags: CODE ANDROID JAVA
finished: true
---


### JSON:

    {
        "name":{
            "firstName":"John",
            "lastName":"Doe",
        }
        "title":"Missing Person"
    }

### Initialize JSON Object from JSON string

    JSONObject contact = new JSONObject(contactJSONString);
    
### Name into a JSON Object

    JSONObject name = contact.getJSONObject("name");
    
### First and last names

    String firstName = name.getString("firstName");
    String lastName = name.getString("lastName");
    
### Title

    String title = contact.getString("title");
    
