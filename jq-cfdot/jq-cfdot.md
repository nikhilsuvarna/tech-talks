

# AGENDA

* **Intro to JSON**
* **jq basics**
* **cfdot**
* **Use Cases** 

---

## Intro to JSON?

-   A JSON string contains either an array of values, or an object (an associative array of name/value pairs).
-   An  _array_  is surrounded by square brackets,  `[`  and  `]`, and contains a comma-separated list of values.
-   An  _object_  is surrounded by curly brackets,  `{`  and  `}`, and contains a comma-separated list of name/value pairs.
-   A  _name/value pair_  consists of a field name (in double quotes), followed by a colon (`:`), followed by the field value.
-   A  _value_  in an array or object can be:  
      
    -   A number (integer or floating point)
    -   A string (in double quotes)
    -   A boolean (`true`  or  `false`)
    -   Another array (surrounded by square brackets,  `[`  and  `]`)
    -   Another object (surrounded by curly brackets,  `{`  and  `}`)
    -   The value  `null`

---
```
{
  "cell_id": "b92b8733-6ca5-4460-80eb-ff908d3456c8",
  "rep_address": "http://10.0.16.18:1800",
  "zone": "z1",
  "capacity": {
    "memory_mb": 26127,
    "disk_mb": 62527,
    "containers": 249
  },
  "rootfs_provider_list": [
    {
      "name": "preloaded",
      "properties": [
        "cflinuxfs2"
      ]
    },
    {
      "name": "preloaded+layer",
      "properties": [
        "cflinuxfs2"
      ]
    },
    {
      "name": "docker"
    }
  ],
  "rep_url": "https://b92b8733-6ca5-4460-80eb-ff908d3456c8.cell.service.cf.internal:1801"
}
```
---
Lets break this down :

-   At the top level, we've written curly braces (`{`  and  `}`), which creates an object.
-   Inside the object, we have several name/value pairs:

| |  |
| :---         |     :---      |  
| `"cell_id": "b92b8733-6ca5-4460-80eb-ff908d3456c8"` | A property with the name `"cell_id"` and the integer value `b92b8733-6ca5-4460-80eb-ff908d3456c8`    |
| `"rootfs_provider_list": [..]`    | A property with the name `"rootfs_provide_list"`, whose value in an array|
|`"capacity": {..}`     | A property with the name `capacity, whose value is an object|

- Inside the `rootfs_provider_list` array, we have 3 properties of the type object which have further nested properties, one is a property with the name `name` and the second, an array which has one value

Review : 
[https://codebeautify.org/jsonviewer/cb1bef4d](https://codebeautify.org/jsonviewer/cb1bef4d)
jqplay


---

##  jq basics

#### jq invocation 

stdin -> `jq [filter]` -> stdout : Read from `stdin` ; write to `stdout`

`curl -s ... | jq ...` 
`cfdot ... | jq ...`

`jq [filter]` file -> stdout ; read from file; write to `stdout`

---

#### Common commandline options

##### compact mode ( `--compact-output` or `-c`)
puts out the result in a compact mode by putting each json object in a single line

`echo '{"x":1, "y":2}' | jq -c`

```
{"x":1,"y":2}
```
##### output raw mode (`--raw-ouput` or `-r`)

without `-r` option :
` echo '{"x":{"y":"z"}}' | jq  .x.y`
```
"z"
```
with `-r` option :
` echo '{"x":{"y":"z"}}' | jq -r .x.y`

```
z
```
Note : there is no `" "` in the result above

##### slurp mode (`--slurp` or `-s`)
smush together the arrays of messages in `1.json` and `2.json` and deal with them as one giant array.
`jq --slurp  . 1.json 2.json`


---

#### Basic Filters

##### Identity filter "jq ."

`echo '{"x":1, "y":2}' | jq .`

```
{
  "x": 1,
  "y": 2
}
```

##### Projection "jq .property"

* Project out the value of x" from `{"x":1, "y":2}`

` echo '{"x":1, "y":2}'|jq .x`
```
1
```

##### Nested Projection "jq. nested.property"

e.g : `'{"x":{"y":"z"}}'`
 
* `z` is a property assigned to object `y` which is assigned to object `x`
* project out the value z by chaining the path expression
```
echo '{"x":{"y":"z"}}' | jq .
{
  "x": {
    "y": "z"
  }
}
```
##### Array 

e.g : `'[{"x":1}, {"y":2}]'`

`echo '[{"x":1}, {"y":2}]' | jq .`
```
[
  {
    "x": 1
  },
  {
    "y": 2
  }
]
```
* Print the contents of the array 

`echo '[{"x":1}, {"y":2}]'| jq .[]`

```
{
  "x": 1
}
{
  "y": 2
}
```

The family of `.[]` is very versatile
  *  If the input is an array,  `.[]`  will iterate through the elements.
  * If the input is an object  `.[]`  will iterate through the object values. for eg : 	
`echo '{"x":{"y":"z"}, "a":{"b":"c"}}'| jq .[]` 
 ```
{
  "y": "z"
}
{
  "b": "c"
} 
```
  * If you provide an index,  `.[0]`  will access elements in arrays by their position.
`echo '[{"x":1}, {"y":2}]'| jq .[0]`
```
{
   "x": 1
}
```

  * If you provide a string,  `.["foo"]`  will access elements in objects by their keys.
  
 
#### Basic Operators

##### pipe
```
jq | filter[1] | filter[2]
```
```
echo '{"x":{"y":"z"}}' | jq .x
{
  "y": "z"
}
```
```
echo '{"x":{"y":"z"}}' | jq '.x | .y'
"z"
```

##### Tee (One record becomes two records)

`echo '{"x":1}'|jq '. , .'`
```
{
  "x": 1
}
{
  "x": 1
}
```

##### Select a value from a key in an object 

`echo '{"foo": 1, "bar": 2}' | jq -c '{foo}'`
```
{"foo":1}
```

##### Build an object out the values of KV pair  
`echo '{"x":{"y":"z"}, "a":{"b":"c"}}'| jq  '{label1: .x.y, label2: .a.b}'`

```
{
  "label1": "z",
  "label2": "c"
}
```
##### add  or `+`
The filter `add` or `+` takes as input, an array and produces an output of the elements of the array added together
* if the elements of the array is an integer, it will be mathematically added

`echo '{"a": 7}' | jq '.a+1'`
```
8
```

* if the objects are mapped to any array, then the result is a concatenation into a larger array

`echo '{"a": [1,2], "b": [3,4]}' | jq -c '.a + .b'`
```
[1,2,3,4]
```
######  map

Print the values of the key/value pair : 
`echo '[{"x":1}, {"y":2}]'| jq '.[]|.[]'`
```
1
2
```
Print the values in an array :

`echo '[{"x":1}, {"y":2}]'| jq '[.[]|.[]]'`

```
[
  1,
  2
]
```
The equivalent of the above command is `map` as show below :

`echo '[{"x":1}, {"y":2}]'| jq 'map(.[])'`
```
[
  1,
  2
]
```
map will run the filter for each element of the input array and returns the output in a new array. In the above example the filter is  `.[]` 	


###### select 

###### group-by

## cfdot

Some common commands :

1) `actual-lrp-groups` : to dump the actual LRPs
2) `actual-lrp-groups-for-guid` :  list actual LRP for a process guid
3) `task-events` : subscribe to task events
4) `lrp-events` : subscribe to lrp events
5) `desired-lrps` : dump desired LRPs
6) `locks` : list locket locks

##### Use Case 1 :

How to find the `instance_guid` for a particular `app_guid`?

a) first find the process guid
```
cf curl /v2/apps/$(cf app <app_name> --guid)| jq -r '.metadata.guid + "-" + .entity.version'
```
https://jqplay.org/s/7kWJ2Hn3UX

b)  identify the diego cell where the app instances are running : 
`cf curl v2/apps/$(cf app <app_name> --guid)/stats | grep host`

c) run `cfdot` on the cell 
```
cfdot actual-lrp-groups-for-guid <process_guid>
```
c) Inspect json blob
[https://codebeautify.org/jsonviewer/cb46f834](https://codebeautify.org/jsonviewer/cb46f834)
d) Use jqplay to start operating on the json blob
https://jqplay.org/s/EB0r_Jz5ay

```
cfdot actual-lrp-groups-for-guid eb2f93fb-b2fc-4f7b-998d-063c18cf8ba0-27cc5569-6f5d-4f47-baf6-5c3d43f0e785| jq '.instance | select (.address == "10.0.16.17")| .instance_guid'
```
d) Once you have the instance guid, then you have backdoor to the container. 

e) optional : List all diego cells with their respective `instance_guid`s for a specific `process-guid`

```
cfdot actual-lrp-groups-for-guid $pg | jq '.instance | "\(.index): Cell \(.cell_id), Instance \(.instance_guid)"' -r
```
##### Use Case 2

For a specific app-guid, find the desired LRP and inspect the json blob. 

The desired LRP tells you a lot of useful things like:

-   environment variables being set
-   the URL from which the droplet is being downloaded
-   the commands being used to run the app
-   app start timeout
-   info about the healthcheck to be used
-   memory & disk limits + CPU weight
-   routes bound & if those are in an isolation segment
-   egress rules
-   the URL from which the buildpack cache can be downloaded

##### Use Case 3

Subscribe to tasks in diego (specifically staging tasks) - This is useful in debugging staging failures


## References :

https://stedolan.github.io/jq/tutorial/
[Playbook](https://github.com/pivotal-gss/pcf-guide/blob/876239c50c4f6a067b948bd8a817fa10fe4e23a3/playbooks/diego/runbooks/useful-cfdot-commands.md) (Credit to Michael Forrest)
[cfdot KB](https://pivotal.lightning.force.com/lightning/r/KnowledgeArticle__kav/ka00P0000009Kq5QAE/view)
https://monades.roperzh.com/weekly-command-processing-json-with-jq/
