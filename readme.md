Mongolite
=========

Streaming Mongo Client for R.

[![Build Status](https://travis-ci.org/jeroenooms/mongolite.svg?branch=master)](https://travis-ci.org/jeroenooms/mongolite)

Install
-------

On most platforms, starting the MongoDB server is as easy as:

```
mongod
```

To install the R package:

```r
library(devtools)
install_github("jeroenooms/jsonlite")
install_github("jeroenooms/mongolite")
```

Hello World
-----------

```r
# Init connection to local mongod
library(mongolite)
m <- mongo(collection = "diamonds")

# Insert test data
data(diamonds, package="ggplot2")
m$insert(diamonds)

# Check records
m$count()
nrow(diamonds)

# Perform a query and retrieve data
out <- m$find('{"cut" : "Premium", "price" : { "$lt" : 1000 } }')

# Compare
nrow(out)
nrow(subset(diamonds, cut == "Premium" & price < 1000))
```

Flights
-------

Some example queries from the dplyr tutorials.

```r
# Insert some data
data(flights, package = "nycflights13")
m <- mongo(collection = "nycflights")
m$insert(flights)

# Basic queries
m$count('{"month":1, "day":1}')
jan1 <- m$find('{"month":1, "day":1}')

# Sorting
jan1 <- m$find('{"$query":{"month":1,"day":1}, "$orderby":{"distance":-1}}')
head(jan1)

# Select columns
jan1 <- m$find('{"month":1,"day":1}', fields = '{"_id":0, "distance":1, "carrier":1}')

# Tabulate
m$aggregate('[{"$group":{"_id":"$carrier", "count": {"$sum":1}, "average":{"$avg":"$distance"}}}]')
```

Combine with jsonlite
---------------------

Example data with zipcodes from [mongolite tutorial](http://docs.mongodb.org/manual/tutorial/aggregation-zip-code-data-set/). This dataset has an `_id` column so you cannot insert it more than once.

```r
library(jsonlite)
library(mongolite)

# Stream from url into mongo
m <- mongo("zips")
stream_in(url("http://media.mongodb.org/zips.json"), handler = function(df){
  m$insert(df, verbose = FALSE)
})

# Check count
m$count()

# Import. Note the 'location' column is actually an array!
zips <- m$find()
```

Nested data
-----------

Stream large bulk samples from [openweathermap](http://openweathermap.org/current#bulk) with deeply nested data (takes a while).

```r
m <- mongo("weather")
stream_in(gzcon(url("http://78.46.48.103/sample/daily_14.json.gz")), handler = function(df){
  m$insert(df, verbose = FALSE)  
}, pagesize = 50)

berlin <- m$find('{"city.name" : "Berlin"}')
print(berlin$data)
```
