---
title: Mongo慢查询
url: 394.html
id: 394
categories:
  - 小提示
date: 2018-11-26 10:37:19
tags:
---

    db.system.profile.find({"op" : "query","ns" : "poetries", millis : { $gt : 100 },ts : { $gt : new ISODate("2018-06-07T00:00:00.000Z"),$lt : new ISODate("2018-06-07T10:00:00.000Z")}}).pretty()
    

    db.getProfilingLevel()   1为开启