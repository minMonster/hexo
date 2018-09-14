---
layout: _post
title: js对象数组(JSON)根据某个共同字段分组
date: 2018-09-14 18:43:55
description: js对象数组(JSON)根据某个共同字段分组
tags: [javaScript, Tool]
categories: javaScript Tool
---
希望的是将下面的对象数组：

```javascript
[
    {"id":"1001","name":"值1","value":"111"},
    {"id":"1001","name":"值1","value":"11111"},
    {"id":"1002","name":"值2","value":"25462"},
    {"id":"1002","name":"值2","value":"23131"},
    {"id":"1002","name":"值2","value":"2315432"},
    {"id":"1003","name":"值3","value":"333333"}
]
```

根据相同id字段分组，转换成下面这种形式：

```JSON
[
    {
        "id": "1001",
        "name": "值1",
        "data": [
            {"id": "1001", "name": "值1", "value": "111"},
            { "id": "1001", "name": "值1", "value": "11111"}
        ]
    },
    {
        "id": "1002",
        "name": "值2",
        "data": [
            { "id": "1002",  "name": "值2", "value": "25462" },
            { "id": "1002", "name": "值2", "value": "23131"},
            {"id": "1002", "name": "值2","value": "2315432" }
        ]
    },
    {
        "id": "1003",
        "name": "值3",
        "data": [
            {"id": "1003", "name": "值3", "value": "333333" }
        ]
    }
]
```

做法：
```javascript
var arr = [
    {"id":"1001","name":"值1","value":"111"},
    {"id":"1001","name":"值1","value":"11111"},
    {"id":"1002","name":"值2","value":"25462"},
    {"id":"1002","name":"值2","value":"23131"},
    {"id":"1002","name":"值2","value":"2315432"},
    {"id":"1003","name":"值3","value":"333333"}
];
​
var map = {},
    dest = [];
for(var i = 0; i < arr.length; i++){
    var ai = arr[i];
    if(!map[ai.id]){
        dest.push({
            id: ai.id,
            name: ai.name,
            data: [ai],
            num: Number(ai.id)
        });
        map[ai.id] = ai;
    }else{
        for(var j = 0; j < dest.length; j++){
            var dj = dest[j];
            if(dj.id == ai.id){
                dj.data.push(ai);
                dj.num += Number(ai.id)
                break;
            }
        }
    }
}
​
console.log(JSON.stringify(dest));
```
```JSON
[
    {
      'id': '1001',
      'name': '值1',
      'data': [{'id': '1001', 'name': '值1', 'value': '111'}, {'id': '1001', 'name': '值1', 'value': '11111'}],
      'num': 2002
    }, {
      'id': '1002',
      'name': '值2',
      'data': [
        {'id': '1002', 'name': '值2', 'value': '25462'}, {
          'id': '1002',
          'name': '值2',
          'value': '23131'
        }, {
          'id': '1002', 
          'name': '值2',
          'value': '2315432'
        }
      ],
      'num': 3006
    }, {
      'id': '1003',
      'name': '值3',
      'data': [{'id': '1003', 'name': '值3', 'value': '333333'}], 
      'num': 1003
    }
]
```
