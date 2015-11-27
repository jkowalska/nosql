###Zadanie 1a

#####Import pliku RC_2015-01 do bazy MongoDB

Zaimportowałam plik korzystając z poniższej komendy:
```sh
time bunzip2 -c RC_2015-01.bz2 | mongoimport --drop --host 127.0.0.1 -d test -c reddit
```
![import](img/obraz1.png)

Historia Procesora:

![procesor](img/obraz2.png)

Procesory były obciążane równomiernie od 25 do 95 procent. Pamięć była wykorzystywana od 28 do 31 procent.

Połączyłam się z mongo, przeszłam do bazy testi wybrałam kolekcję reddit:
```sh
mongo
MongoDB shell version: 2.6.3
connecting to: test
> show dbs
local	0.203125GB
test	37.935546875GB
> use test
switched to db test
> show collections
reddit
system.indexes
```
Policzyłam wszystkie jsony:

![json](img/obraz3.png)

Przykładowe zapytania:
* znajdź pierwsze:
```sh
> db.reddit.findOne()
{
	"_id" : ObjectId("5657848cc40dd605ebeb4d7d"),
	"score_hidden" : false,
	"name" : "t1_cnas8zv",
	"link_id" : "t3_2qyr1a",
	"body" : "Most of us have some family members like this. *Most* of my family is like this. ",
	"downs" : 0,
	"created_utc" : "1420070400",
	"score" : 14,
	"author" : "YoungModern",
	"distinguished" : null,
	"id" : "cnas8zv",
	"archived" : false,
	"parent_id" : "t3_2qyr1a",
	"subreddit" : "exmormon",
	"author_flair_css_class" : null,
	"author_flair_text" : null,
	"gilded" : 0,
	"retrieved_on" : 1425124282,
	"ups" : 14,
	"controversiality" : 0,
	"subreddit_id" : "t5_2r0gj",
	"edited" : false
}
> 
```
*znajdź ostatnie:
```sh
> db.reddit.findOne( {$query:{}, $orderby:{$natural:-1}} )
{
	"_id" : ObjectId("56579d9ec40dd605eb210312"),
	"author_flair_text" : "RRRAURGH!",
	"gilded" : 0,
	"score_hidden" : false,
	"id" : "co77gzt",
	"parent_id" : "t1_co6zqmw",
	"distinguished" : null,
	"ups" : 3,
	"downs" : 0,
	"created_utc" : "1422748799",
	"name" : "t1_co77gzt",
	"body" : "You can already shoot through walls without Shred.",
	"author_flair_css_class" : "valkyr-bastet",
	"subreddit_id" : "t5_2urg0",
	"link_id" : "t3_2uazsm",
	"controversiality" : 0,
	"edited" : false,
	"retrieved_on" : 1424281770,
	"author" : "blolfighter",
	"subreddit" : "Warframe",
	"archived" : false,
	"score" : 3
}
> 
```

