#####Import pliku bazy danych restaurants.json do bazy MongoDB wersja 3.0.8

Pobrałam plik bazy **Restaurants** z brytyjskimi restauracjami wielkości 684,2 kB. ze strony [restaurants.json](https://dl.dropboxusercontent.com/u/15056258/mongodb/restaurant.json).

Zaimportowałam go do bazy MongoDB korzystając z poniższej komendy:
```sh
time mongoimport -d restaurants -c restaurants < restaurants.json

connected to: 127.0.0.1
2016-01-10T11:42:17.126+0100	connected to: localhost
2016-01-10T11:42:17.728+0100	imported 2548 documents

real	0m0.619s
user	0m0.158s
sys     0m0.014s
```
* przykładowy json z kolekcji (ostatni) wraz ze wszystkimi rekordami: id, url, adresem, nazwą, kodami pocztowymi, oceną i typem jedzenia:
```sh
db.restaurants.findOne( {$query:{}, $orderby:{$natural:-1}} )
{
  "_id": ObjectId("55f14313c7447c3da7052519"),
  "URL": "http://www.just-eat.co.uk/restaurants-bluebreeze-le3/menu",
  "address": "56 Bonney Road",
  "address line 2": "Leicester",
  "name": "Blue Breeze Fish Bar",
  "outcode": "LE3",
  "postcode": "9NG",
  "rating": 5.5,
  "type_of_food": "Fish & Chips"
}
```
* połączyłam się z mongo i przeszłam do bazy restaurants:
```sh
mongo
MongoDB shell version: 3.0.8
use restaurants
switched to db restaurants
restaurants> 
```
* policzyłam wszystkie jsony (zaimportowały się poprawnie):
```sh
db.restaurants.count()
2548
```
* wyświetlenie nazw i oceny 5 restauracji zaczynających się na "Bis"
```sh
db.restaurants.find({"name": /^Bis/}, {_id: 0, name: 1, rating: 1, type_of_food: 1}).sort({name: 1}).limit(5)
{
  "name": "Bishaal",
  "rating": 5.5,
  "type_of_food": "Curry"
}
{
  "name": "Bishops Caribbean",
  "rating": 5.5,
  "type_of_food": "Caribbean"
}
{
  "name": "Bishy's Chicken",
  "rating": 4,
  "type_of_food": "American"
}
{
  "name": "Bistro 5 Pizzeria",
  "rating": 4.5,
  "type_of_food": "Pizza"
}
{
  "name": "Bistro Chicken",
  "rating": 5,
  "type_of_food": "Chicken"
}
Fetched 5 record(s) in 8ms
```
* wyświetlenie rodzajów podawanego jedzenia w restauracjach (posortowane):
```sh
db.restaurants.distinct("type_of_food").sort()
[
  "*NEW*",
  "Afghan",
  "African",
  "American",
  "Arabic",
  "Azerbaijan",
  "Bagels",
  "Bangladeshi",
  "Breakfast",
  "Burgers",
  "Cakes",
  "Caribbean",
  "Chicken",
  "Chinese",
  "Curry",
  "Desserts",
  "English",
  "Ethiopian",
  "Fish & Chips",
  "Greek",
  "Grill",
  "Healthy",
  "Ice Cream",
  "Japanese",
  "Kebab",
  "Korean",
  "Lebanese",
  "Mediterranean",
  "Mexican",
  "Middle Eastern",
  "Milkshakes",
  "Moroccan",
  "Nigerian",
  "Pakistani",
  "Pasta",
  "Peri Peri",
  "Persian",
  "Pick n Mix",
  "Pizza",
  "Polish",
  "Portuguese",
  "Punjabi",
  "Russian",
  "Sandwiches",
  "South Curry",
  "Spanish",
  "Sri-lankan",
  "Sushi",
  "Thai",
  "Turkish",
  "Vegetarian",
  "Vietnamese"
]
```
* wyświetlenie restauracji podających wietnamskie jedzenie:
```sh
db.restaurants.find({type_of_food: "Vietnamese"},{_id: 0})
{
  "URL": "http://www.just-eat.co.uk/restaurants-anhdao/menu",
  "address": "106-108 Kingsland Road",
  "address line 2": "London",
  "name": "Anh Dao",
  "outcode": "E2",
  "postcode": "8DP",
  "rating": 4,
  "type_of_food": "Vietnamese"
}
{
  "URL": "http://www.just-eat.co.uk/restaurants-anhdao/menu",
  "address": "106-108 Kingsland Road",
  "address line 2": "London",
  "name": "Anh Dao",
  "outcode": "E2",
  "postcode": "8DP",
  "rating": 4,
  "type_of_food": "Vietnamese"
}
Fetched 2 record(s) in 6ms
```
###Agregacje:

* wyświetlenie sum oceny typów jedzenia dla oceny >= 50 i < 100 (posortowane od największej do najmniejszej sumy ocen):
```sh
db.restaurants.aggregate([
  { $group: {_id: "$type_of_food", totalRating: {$sum: "$rating"}} },
  { $match: {totalRating: {$gte: 50, $lt: 100}} },
  { $sort: {totalRating: -1 } }
])
{
  "result": [
    {
      "_id": "Bangladeshi",
      "totalRating": 95.5
    },
    {
      "_id": "Peri Peri",
      "totalRating": 92.5
    },
    {
      "_id": "African",
      "totalRating": 91
    },
    {
      "_id": "Japanese",
      "totalRating": 82
    },
    {
      "_id": "Afghan",
      "totalRating": 74.5
    },
    {
      "_id": "Persian",
      "totalRating": 73.5
    },
    {
      "_id": "Middle Eastern",
      "totalRating": 63.5
    },
    {
      "_id": "Mediterranean",
      "totalRating": 62
    },
    {
      "_id": "Grill",
      "totalRating": 58.5
    },
    {
      "_id": "Mexican",
      "totalRating": 52.5
    }
  ],
  "ok": 1
}
```
* wyświetlenie średnich ocen dla wszystkich typów jedzenia posortowane według średniej oceny jedzenia:
```sh
db.restaurants.aggregate([
  { $group: {_id: "$type_of_food", avgRating: {$avg: "$rating"}} },
  { $sort: {avgRating: -1 } }
])
{
  "result": [
    {
      "_id": "Pasta",
      "avgRating": 6
    },
    {
      "_id": "Punjabi",
      "avgRating": 6
    },
    {
      "_id": "Bagels",
      "avgRating": 5.5
    },
    {
      "_id": "Ice Cream",
      "avgRating": 5.5
    },
    {
      "_id": "Pick n Mix",
      "avgRating": 5.5
    },
    {
      "_id": "Cakes",
      "avgRating": 5.5
    },
    {
      "_id": "Bangladeshi",
      "avgRating": 5.305555555555555
    },
    {
      "_id": "Persian",
      "avgRating": 5.25
    },
    {
      "_id": "Polish",
      "avgRating": 5.166666666666667
    },
    {
      "_id": "Mediterranean",
      "avgRating": 5.166666666666667
    },
    {
      "_id": "Greek",
      "avgRating": 5.142857142857143
    },
    {
      "_id": "Sushi",
      "avgRating": 5.125
    },
    {
      "_id": "Fish & Chips",
      "avgRating": 5.036697247706422
    },
    {
      "_id": "Curry",
      "avgRating": 5.0361581920903955
    },
    {
      "_id": "Azerbaijan",
      "avgRating": 5
    },
    {
      "_id": "Ethiopian",
      "avgRating": 5
    },
    {
      "_id": "Korean",
      "avgRating": 5
    },
    {
      "_id": "Portuguese",
      "avgRating": 5
    },
    {
      "_id": "Breakfast",
      "avgRating": 5
    },
    {
      "_id": "Afghan",
      "avgRating": 4.966666666666667
    },
    {
      "_id": "Burgers",
      "avgRating": 4.954545454545454
    },
    {
      "_id": "Turkish",
      "avgRating": 4.918918918918919
    },
    {
      "_id": "Pizza",
      "avgRating": 4.914141414141414
    },
    {
      "_id": "Chinese",
      "avgRating": 4.89367816091954
    },
    {
      "_id": "Kebab",
      "avgRating": 4.88562091503268
    },
    {
      "_id": "Grill",
      "avgRating": 4.875
    },
    {
      "_id": "Peri Peri",
      "avgRating": 4.868421052631579
    },
    {
      "_id": "South Curry",
      "avgRating": 4.833333333333333
    },
    {
      "_id": "Japanese",
      "avgRating": 4.823529411764706
    },
    {
      "_id": "Lebanese",
      "avgRating": 4.8059701492537314
    },
    {
      "_id": "Milkshakes",
      "avgRating": 4.8
    },
    {
      "_id": "Mexican",
      "avgRating": 4.7727272727272725
    },
    {
      "_id": "Sri-lankan",
      "avgRating": 4.7
    },
    {
      "_id": "Moroccan",
      "avgRating": 4.666666666666667
    },
    {
      "_id": "Sandwiches",
      "avgRating": 4.666666666666667
    },
    {
      "_id": "Thai",
      "avgRating": 4.65
    },
    {
      "_id": "American",
      "avgRating": 4.617021276595745
    },
    {
      "_id": "Caribbean",
      "avgRating": 4.583333333333333
    },
    {
      "_id": "Middle Eastern",
      "avgRating": 4.535714285714286
    },
    {
      "_id": "Nigerian",
      "avgRating": 4.5
    },
    {
      "_id": "Russian",
      "avgRating": 4.5
    },
    {
      "_id": "Spanish",
      "avgRating": 4.5
    },
    {
      "_id": "Arabic",
      "avgRating": 4.5
    },
    {
      "_id": "English",
      "avgRating": 4.5
    },
    {
      "_id": "Chicken",
      "avgRating": 4.41
    },
    {
      "_id": "Desserts",
      "avgRating": 4.375
    },
    {
      "_id": "Pakistani",
      "avgRating": 4.357142857142857
    },
    {
      "_id": "Healthy",
      "avgRating": 4.166666666666667
    },
    {
      "_id": "Vietnamese",
      "avgRating": 4
    },
    {
      "_id": "Vegetarian",
      "avgRating": 4
    },
    {
      "_id": "African",
      "avgRating": 3.64
    },
    {
      "_id": "*NEW*",
      "avgRating": 0
    }
  ],
  "ok": 1
}
```
* wyświetlenie 10 typów jedzenia występujących najczęściej:
```sh
db.restaurants.aggregate([
  {"$group" : {"_id" : "$type_of_food", "count" : {"$sum" : 1}}},
  {"$sort" : {"count" : -1}},
  {"$limit" : 10}])
{
  "result": [
    {
      "_id": "Curry",
      "count": 902
    },
    {
      "_id": "Pizza",
      "count": 500
    },
    {
      "_id": "Chinese",
      "count": 174
    },
    {
      "_id": "Kebab",
      "count": 154
    },
    {
      "_id": "Fish & Chips",
      "count": 116
    },
    {
      "_id": "American",
      "count": 95
    },
    {
      "_id": "Turkish",
      "count": 74
    },
    {
      "_id": "Lebanese",
      "count": 70
    },
    {
      "_id": "Chicken",
      "count": 53
    },
    {
      "_id": "Caribbean",
      "count": 46
    }
  ],
  "ok": 1
}
```
* wyświetlenie 10 typów jedzenia występujących najrzadziej:
```sh
db.restaurants.aggregate([
  {"$group" : {"_id" : "$type_of_food", "count" : {"$sum" : 1}}},
  {"$sort" : {"count" : 1}},
  {"$limit" : 10}])
{
  "result": [
    {
      "_id": "Pasta",
      "count": 1
    },
    {
      "_id": "Cakes",
      "count": 1
    },
    {
      "_id": "Punjabi",
      "count": 1
    },
    {
      "_id": "Spanish",
      "count": 1
    },
    {
      "_id": "Nigerian",
      "count": 1
    },
    {
      "_id": "Pick n Mix",
      "count": 2
    },
    {
      "_id": "Vietnamese",
      "count": 2
    },
    {
      "_id": "Portuguese",
      "count": 2
    },
    {
      "_id": "Russian",
      "count": 2
    },
    {
      "_id": "Vegetarian",
      "count": 2
    }
  ],
  "ok": 1
}
```
