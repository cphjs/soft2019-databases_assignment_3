#



# Twitter

## How many Twitter users are in the database?

### Query

```
db.tweets.aggregate([ 
    { $group: { _id: '$user' } }, 
    { $group: { _id: null, count: { $sum: 1}  } } 
]);
```

### Results

```
{ "_id" : null, "count" : 659774 }
```


## Which Twitter users link the most to other Twitter users?

### Query

```
db.tweets.aggregate([ 
    { $match: { text: { $regex: /@\w+/g} } }, 
    { $group: { _id: '$user', 'links': { $sum: 1} } }, 
    { $sort: {links: -1} },
    { $limit: 10 }
]);
```

### Results

```
{ "_id" : "lost_dog", "links" : 549 }
{ "_id" : "tweetpet", "links" : 310 }
{ "_id" : "VioletsCRUK", "links" : 251 }
{ "_id" : "what_bugs_u", "links" : 246 }
{ "_id" : "tsarnick", "links" : 245 }
{ "_id" : "SallytheShizzle", "links" : 229 }
{ "_id" : "mcraddictal", "links" : 217 }
{ "_id" : "Karen230683", "links" : 216 }
{ "_id" : "keza34", "links" : 211 }
{ "_id" : "TraceyHewins", "links" : 202 }
```

## Who is are the most mentioned Twitter users?

### Query

```
db.tweets.aggregate([ 
    { $addFields: { words: { $split: ['$text', ' ']}  } }, 
    { $unwind: '$words' }, 
    { $match: { words: { $regex: /@\w+/ } }  }, 
    { $group: { _id: '$words', count: { $sum: 1} }  }, 
    { $sort: { count: -1 } }, 
    { $limit: 10 }
]);
```

### Results
```
{ "_id" : "@mileycyrus", "count" : 4310 }
{ "_id" : "@tommcfly", "count" : 3837 }
{ "_id" : "@ddlovato", "count" : 3349 }
{ "_id" : "@Jonasbrothers", "count" : 1263 }
{ "_id" : "@DavidArchie", "count" : 1222 }
{ "_id" : "@jordanknight", "count" : 1105 }
{ "_id" : "@DonnieWahlberg", "count" : 1085 }
{ "_id" : "@JonathanRKnight", "count" : 1053 }
{ "_id" : "@mitchelmusso", "count" : 1038 }
{ "_id" : "@taylorswift13", "count" : 973 }
```

## Who are the most active Twitter users?

### Query 

```
db.tweets.aggregate([
    { "$group": { 
            "_id": "$user",
            "count": { "$sum":  1},
        }
    },
    { "$sort": { "count": -1 } },
    { "$limit": 10 }
]);
```

### Results

```
{ "_id" : "lost_dog", "count" : 549 }
{ "_id" : "webwoke", "count" : 345 }
{ "_id" : "tweetpet", "count" : 310 }
{ "_id" : "SallytheShizzle", "count" : 281 }
{ "_id" : "VioletsCRUK", "count" : 279 }
{ "_id" : "mcraddictal", "count" : 276 }
{ "_id" : "tsarnick", "count" : 248 }
{ "_id" : "what_bugs_u", "count" : 246 }
{ "_id" : "Karen230683", "count" : 238 }
{ "_id" : "DarkPiano", "count" : 236 }
```
## Who are the five most grumpy (most negative tweets) and the most happy (most positive tweets)?

### Queries

```
db.tweets.aggregate([
    { $match: { polarity: 0 } },
    { 
        $group: {
            _id: { user: '$user', polarity: '$polarity'},
            count: { $sum: 1 },
        }
    },
    { $sort: { count: -1 } },
    { $limit: 5 },
]);
```


```
db.tweets.aggregate([
    { $match: { polarity: 4 } },
    { 
        $group: {
            _id: { user: '$user', polarity: '$polarity'},
            count: { $sum: 1 },
        }
    },
    { $sort: { count: -1 } },
    { $limit: 5 },
]);
```

### Results


```
{ "_id" : { "user" : "lost_dog", "polarity" : 0 }, "count" : 549 }
{ "_id" : { "user" : "tweetpet", "polarity" : 0 }, "count" : 310 }
{ "_id" : { "user" : "webwoke", "polarity" : 0 }, "count" : 264 }
{ "_id" : { "user" : "mcraddictal", "polarity" : 0 }, "count" : 210 }
{ "_id" : { "user" : "wowlew", "polarity" : 0 }, "count" : 210 }
```

```
{ "_id" : { "user" : "what_bugs_u", "polarity" : 4 }, "count" : 246 }
{ "_id" : { "user" : "DarkPiano", "polarity" : 4 }, "count" : 231 }
{ "_id" : { "user" : "VioletsCRUK", "polarity" : 4 }, "count" : 218 }
{ "_id" : { "user" : "tsarnick", "polarity" : 4 }, "count" : 212 }
{ "_id" : { "user" : "keza34", "polarity" : 4 }, "count" : 211 }
```

# Modeling

Please do keep in mind that I have missed this lecture and might have no idea what I am doing. Therefore any insights are appreciated :)

Model | Atomicity | Sharding |Indexes |Large Number of Collections | Collection Contains Large Number of Small Documents
  ----|:----:|:----:|:----:|:----:|:----:
Arrays of Ancestors	|Related documents;no atomicity between documents, only within| |Requires indexing an array, which will take significant resources|I cannot think of a way to split a collection using this pattern into multiple| |
Materialized paths  | |Find queries would be broadcasted to all shards||Each subpath could probably be a collection. For example instead of having a single collection called `categories` for all data you could have `categories_books`, `categeories_books_programming` and so on.|Small documents can be easily "rolled up" into bigger ones by using the path. In that case the separator in a material path could become a level of nesting|
Nested sets			|Same as Array of Ancestors|Find queries can be targeted to specific shards|Simple index, should be good|| |


