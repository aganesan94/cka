# JSON Paths

* Used to query json

### Querying from Root



* Use a $ notation to start from the root element
* All of the below uses jpath

```json
{
  "car": {
    "color": "blue",
    "price: "$20000"
  },
  "bus": {
    "color": "white",
    "price: "$120000"
  }
}
```

Query: 
```json
$.vehicle.car
```

Result:

```json
{
  "color": "blue",
  "price": "$20000"
}
```
---

### Querying by index

```json
[
    "car",
    "bus",
    "truck",
    "bike"
]
```

Query and result:

```json
  $[0] - ["car"]
  $[3] - ["bike"]
  $[0,3] - ["car","bike"]
  $[0:2] - ["car","bus","truck"]
  $[0:2:2] - ["car","truck"]
  $[-1] - last index in the array
  $[-1:0] - start from the last element all the way to the end.
  $[-3:0] - last 3 elements , same as $[-3] or $[-3:]
  $[0,2] - ["car","truck"]
```

---

```json
[
  12,
  43,
  23,
  12,
  56
]
```

Query: Get all items in the array which are above for

```text
$[?( @ > 40 )]

# Other queries
@ == 40
@ != 40
@ in [40,43,45]
@ nin [40,43,45]

Returns model whose location is rear-right
$.car.wheels[? (@.location == "rear-right" ) ].model 
```

---

### Querying All

```json
{
  "car": {
    "wheels": [
      {
        "model": "x"
      },
      {
        "model": "y"
      }
    ]
  },
  "bus" : {
    "wheels": [
      {
        "model": "x"
      },
      {
        "model": "y"
      }
  }
}
```

Query

```text
# To get all the models across all the wheels
$.*.wheels[*]

# To get all the models for a car wheel
$.car.wheels[*]
```