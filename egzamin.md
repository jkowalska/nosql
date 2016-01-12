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
sys   0m0.014s
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
* wyświetlenie rodzajów serwowanego jedzenia w restauracjach (posortowane):
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
