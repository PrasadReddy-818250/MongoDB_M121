# MongoDB_M121
Study_Note_&amp;_Solutions

## Chapter 0
**Connect to M121 course Atlas Cluster**
```
mongo "mongodb://cluster0-shard-00-00-jxeqq.mongodb.net:27017,cluster0-shard-00-01-jxeqq.mongodb.net:27017,cluster0-shard-00-02-jxeqq.mongodb.net:27017/aggregations?replicaSet=Cluster0-shard-0" --authenticationDatabase admin --ssl -u m121 -p aggregations --norc
```
**Question**: The Concept of Pipelines
Which of the following is true about pipelines and the Aggregation Framework?

- [x] Documents flow through the pipeline, passing from one stage to the next
- [x] The Aggregation Framework provides us many stages to filter and transform our data
- [ ] Pipeline must consist of at least two stages
- [ ] Stages cannot be configured to produce our desired output

**Aggregation Struture and Syntax**

```
db.solarSystem.aggregate(
  [
    {
      $match:{
        atmosphericComposition: { $in: [/02/] },
        meanTemperature: { $gte: -40, $lte: 40}
      }
    },
    {
      $project: {
        _id: 0,
        name: 1,
        hasMoons: { $gt: [ "$numberOfMoons", 0 ]}
      }
    }
  ]
)
```
**Aggregation Structure and Syntax Rules**
- Pipelines are always an array of one or more stages
```
db.userColl.aggregate([{stage1}, {stage2}, {...stageN}], {options})
```
- Stages are composed of one or more aggregation operators or expressions
- Expressions may take a single argument or an array of arguments. This is expression dependant


## Chapter 1
**Filtering Documents with $match**

Link:[$match](https://university.mongodb.com/mercury/M121/2021_December_21/chapter/Chapter_1_Basic_Aggregation_-_match_and_project/lesson/59dbb62fe433135c3d5b858d/lecture#:~:text=Download%20Course%20Materials-,%24match,-documentation%20page.)
- $match uses standard MongoDB query operators
- $match does not allow for projection
- $match should come early in an aggregation pipeline
- You cannot use $where with $match
- $match uses the same query syntax as find

**Problem** Which of the following is/are true of the $match stage?

- [ ] $match can only filter documents on one field.
- [x] It uses the familiar MongoDB query language.
- [x] It should come very early in an aggregation pipeline.
- [ ] $match can use both query operators and aggregation expressions.

**Lab -$match**

Hint
```
var pipeline = [
  {
    $match: {
      $and: [
        { "imdb.rating": { $gte: 7 } },
        { genres: { $nin: ["Crime", "Horror"] } },
        { rated: { $in: ["P", "PG"] } },
        { languages: { $all: ["English", "Japanese"] } }
      ]
    }
  }
]
```
```
db.movies.aggregate(pipeline).itcount()
```
Answer: 15

**Shaping Documents with $project**

Reference Link: [$project](https://university.mongodb.com/mercury/M121/2021_December_21/chapter/Chapter_1_Basic_Aggregation_-_match_and_project/lesson/59dbc84de433136136d03942/lecture#:~:text=Download%20Course%20Materials-,%24project,-documentation%20page.)

- Once we specify one field to retain, we must specify all fields we want to retain. The _id field is the only exception to this.
- Beyond simply removing and retaining fields, $project lets us add new fields.
- $project can be used as many times as required within an Aggregation pipeline.
- $project can be used to reassign values to existing field names and to derive entirely new fields

**Problem** Which of the following statements are true of the $project stage?

- [x] Once we specify a field to retain or perform some computation in a $project stage, we must specify all fields we wish to retain. The only exception to this is the _id field.

- [x] Beyond simply removing and retaining fields, $project lets us add new fields.

- [ ] $project can only be used once within an Aggregation pipeline.

- [ ] $project cannot be used to assign new values to existing fields.

**Lab - Changing Document Shape with $project**
```
var pipeline = [
  {
    $match: {
      $and: [
        { 'imdb.rating': { $gte: 7 } },
        { genres: { $nin: ['Crime', 'Horror'] } },
        { rated: { $in: ['P', 'PG'] } },
        { languages: { $all: ['English', 'Japanese'] } }
      ]
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      rated: 1
    }
  }
]
```
```
db.movies.aggregate(pipeline).itcount()
```
Answer: 15

**Lab - Computing Fields**

```
db.movies.aggregate([
  {
    $match: {
      title: {
        $type: "string"
      }
    }
  },
  {
    $project: {
      title: { $split: ["$title", " "] },
      _id: 0
    }
  },
  {
    $match: {
      title: { $size: 1 }
    }
  }
]).itcount()
```
Answer: 8066

**Optional Lab - Expressions with $project**

```
db.movies.aggregate([
  {
    $match: {
      cast: { $elemMatch: { $exists: true } },
      directors: { $elemMatch: { $exists: true } },
      writers: { $elemMatch: { $exists: true } }
    }
  },
  {
    $project: {
      _id: 0,
      cast: 1,
      directors: 1,
      writers: {
        $map: {
          input: "$writers",
          as: "writer",
          in: {
            $arrayElemAt: [
              {
                $split: ["$$writer", " ("]
              },
              0
            ]
          }
        }
      }
    }
  },
  {
    $project: {
      labor_of_love: {
        $gt: [
          { $size: { $setIntersection: ["$cast", "$directors", "$writers"] } },
          0
        ]
      }
    }
  },
  {
    $match: { labor_of_love: true }
  },
  {
    $count: "labors of love"
  }
])
```

Answer: 1596

## Chapter 2

Reference Link: [$addFields](https://docs.mongodb.com/manual/reference/operator/aggregation/addFields?jmp=university)

```
db.solarSystem.aggregate(
  [
    {
      $project: {
        _id: 0,
        name: 1,
        gravity: "gravity.value",
        meanTemperature: 1,
        density: 1,
        mass: "$mass.value",
        radius: "$radius.value",
        sma: "$sma.value"
      }
    }
  ]
).pretty()
```
$add Fields
```
db.solarSystem.aggregate(
  [
    {
      $addFields: {
        gravity: "$gravity.value",
        mass: "$mass.value",
        radius: "$radius.value",
        sma: "$sma.value"
      }
    }
   ]
)
```

By using $addFields, it does not remove any fields from the origional document, instead, append new transformation fields to the document.

```
db.solarSystem.aggregate(
  [
    {
      $project: {
        _id: 0,
        name: 1,
        gravity: 1,
        mass: 1,
        radius: 1,
        sma: 1
      }
    }, {
      $addFields: {
        gravity: "$gravity.value",
        mass: "$mass.value",
        radius: "$radius.value",
        sma: "$sma.value"
      }
    }
  ]
).pretty()
```
This is a style choice when performing many various calculations.

**$geoNear**

Reference Link: [$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear?jmp=university)

- $geoNear is the first stage in a pipeline.

**cursor-like stages**
```
db.solarSystem.aggregate(
  [
    {
      "$project": {
        _id: 0,
        name: 1,
        numberOfMoons: 1
      }
    }, {
      "$Limit": 5
    }
  ]
)
```
- {$limit: Integer}
- {$skip: Integer}
- {$count: "field"}
- {$sort: {field1 :-1, field2: 1}}
- By default, $sort will only use up to 100 megabytes of RAM. Setting a allowDiskUse: true will allow for larger sorts.

**$sample**

Reference Link: [$sample](https://docs.mongodb.com/manual/reference/operator/aggregation/sample?jmp=university)

**Lab: Using Cursor-like Stages**

Problem:
MongoDB has another movie night scheduled. This time, we polled employees for their favorite actress or actor, and got these results
For movies released in the USA with a tomatoes.viewer.rating greater than or equal to 3, calculate a new field called num_favs that represets how many favorites appear in the cast field of the movie.
Sort your results by num_favs, tomatoes.viewer.rating, and title, all in descending order.
What is the title of the 25th film in the aggregation result?

```
var favorites = [
  "Sandra Bullock",
  "Tom Hanks",
  "Julia Roberts",
  "Kevin Spacey",
  "George Clooney"]

db.movies.aggregate([
  {
    $match: {
      "tomatoes.viewer.rating": { $gte: 3 },
      countries: "USA",
      cast: {
        $in: favorites
      }
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      "tomatoes.viewer.rating": 1,
      num_favs: {
        $size: {
          $setIntersection: [
            "$cast",
            favorites
          ]
        }
      }
    }
  },
  {
    $sort: { num_favs: -1, "tomatoes.viewer.rating": -1, title: -1 }
  },
  {
    $skip: 24
  },
  {
    $limit: 1
  }
])
```
Answer: The Heat

**Lab - Bringing it all together**

Problem:

Calculate an average rating for each movie in our collection where English is an available language, the minimum imdb.rating is at least 1, the minimum imdb.votes is at least 1, and it was released in 1990 or after. You'll be required to rescale (or normalize) imdb.votes. The formula to rescale imdb.votes and calculate normalized_rating is included as a handout.

What film has the lowest normalized_rating?

```
db.movies.aggregate([
  {
    $match: {
      year: { $gte: 1990 },
      languages: { $in: ["English"] },
      "imdb.votes": { $gte: 1 },
      "imdb.rating": { $gte: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      "imdb.rating": 1,
      "imdb.votes": 1,
      normalized_rating: {
        $avg: [
          "$imdb.rating",
          {
            $add: [
              1,
              {
                $multiply: [
                  9,
                  {
                    $divide: [
                      { $subtract: ["$imdb.votes", 5] },
                      { $subtract: [1521105, 5] }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    }
  },
  { $sort: { normalized_rating: 1 } },
  { $limit: 1 }
])
```
The Answer: The Christmas Tree

## Chapter 3

Reference Link [$group](https://docs.mongodb.com/manual/reference/operator/aggregation/group?jmp=university)


```
db.movies.aggregate(
	[
		{
			$match: { metacritic:{$gte:0}}
		},
		{
			$group:{
				_id:{
					numDirectors:{
						$cond:[{ $isArray:"$directors"},{$size:"$directors"},0]
					} //To create new field
				},
				numFilms: {$sum:1},
				averageMetacritic:{$avg:"$metacritic"}
			}
		},
		{
			$sort:{"_id.numDirecctors":-1}
		}
	]
)
```
$project
```
// run to get a view of the document schema
db.icecream_data.findOne()

// using $reduce to get the highest temperature
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "max_high": {
        "$reduce": {
          "input": "$trends",
          "initialValue": -Infinity,
          "in": {
            "$cond": [
              { "$gt": ["$$this.avg_high_tmp", "$$value"] },
              "$$this.avg_high_tmp",
              "$$value"
            ]
          }
        }
      }
    }
  }
])

// performing the inverse, grabbing the lowest temperature
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "min_low": {
        "$reduce": {
          "input": "$trends",
          "initialValue": Infinity,
          "in": {
            "$cond": [
              { "$lt": ["$$this.avg_low_tmp", "$$value"] },
              "$$this.avg_low_tmp",
              "$$value"
            ]
          }
        }
      }
    }
  }
])

// note that these two operations can be done with the following operations can
// be done more simply. The following two expressions are functionally identical

db.icecream_data.aggregate([
  { "$project": { "_id": 0, "max_high": { "$max": "$trends.avg_high_tmp" } } }
])

db.icecream_data.aggregate([
  { "$project": { "_id": 0, "min_low": { "$min": "$trends.avg_low_tmp" } } }
])

// getting the average and standard deviations of the consumer price index
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "average_cpi": { "$avg": "$trends.icecream_cpi" },
      "cpi_deviation": { "$stdDevPop": "$trends.icecream_cpi" }
    }
  }
])

// using the $sum expression to get total yearly sales
db.icecream_data.aggregate([
  {
    "$project": {
      "_id": 0,
      "yearly_sales (millions)": { "$sum": "$trends.icecream_sales_in_millions" }
    }
  }
])
```

**Lab - $group and Accumulators**

```
db.movies.aggregate(
  [
    {
      $match:{awards: /Won \d{1,2} Oscars?/}
    },
    {
      "$group": {
		_id:null,
		"sd": {"$stdDevPop": "$imdb.rating"},
		"highest": {"$max": "$imdb.rating"},
		"lowest": {"$min": "$imdb.rating"},
		"average":{"$avg": "$imdb.rating"}
		}
    }
  ]
)
```

**$unwind**

Reference Link: [$unwind](https://docs.mongodb.com/manual/reference/operator/aggregation/unwind/)

```
// finding the top rated genres per year from 2010 to 2015...
db.movies.aggregate([
  {
    "$match": {
      "imdb.rating": { "$gt": 0 },
      "year": { "$gte": 2010, "$lte": 2015 },
      "runtime": { "$gte": 90 }
    }
  },
  {
    "$unwind": "$genres"
  },
  {
    "$group": {
      "_id": {
        "year": "$year",
        "genre": "$genres"
      },
      "average_rating": { "$avg": "$imdb.rating" }
    }
  },
  {
    "$sort": { "_id.year": -1, "average_rating": -1 }
  }
])

```

**Lab - $unwind**

```
db.movies.aggregate([
  {
    "$match": {
      "languages": { "$in": ["English"] }
    }
  },
  {
    "$unwind": "$cast"
  },
  {
    "$group": {
      "_id": "$cast",
      "numFilms": {"$sum": 1},
      "average": { "$avg": "$imdb.rating" }
    }
  },
  {
    "$sort": { "numFilms": -1}
  }
])
```
