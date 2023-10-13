# Домашнее задание по 3му уроку
> *Примечание: Работы производились на macOS 14.0 c Docker Desktop 4.24.0 (122432)*

## Установка mongoDB в docker

Запушен образ 
    
    docker pull mongodb/mongodb-community-server
Поднят контейнер с пробросом порта 27017 (по умолчанию порт закрыт)
    
    docker run --name mongo -p 27017:27017 -d mongodb/mongodb-community-server:latest

## Работа с MongoDB
Для работы с MongoDB,установелен mongosh через Homebrew
    
    brew install mongosh
Подключение через mongosh

    mongosh mongosh -port 27017
Пероверка работы через команду вывода баз (по усолчанию созданы будет 3 базы)

    show dbs
*Результат*
```
admin    8.00 KiB
config  12.00 KiB
local    8.00 KiB
```

### Создание новой базы, коллекции и документов
> *Примечание: Если в базу не добавлена хотя бы одна коллекция, а в ней нет хотя бы одного документа, то база не будет создана, что видно при повторном вызове списка баз, после создания базы «testBD»*

    use testBD
*Результат*
```
switched to db testBD
```

А так же после создания коллекции «test_collection»
    
    db.getCollection('test_collection')
*Результат*
```
testBD.test_collection
```

После создания в базе коллекции и 1 документа, база появилась в списке.
    
    db.getCollection('test_collection').insertOne({ name: "Alice", age: 25, grade: "A" })
```
{
  acknowledged: true,
  insertedId: ObjectId("65291c360671f21914043798")
}
```
    
    show dbs
```
admin   40.00 KiB
config  72.00 KiB
local   40.00 KiB
testBD   8.00 KiB
```
Создадим несколько документов через «insertMany»

    db.getCollection('test_collection').insertMany([ { name: "Boris", age: 29, gender: "male", grade: "B" }, { name: "Alex", age: 19, gender: "male", grade: "C" }, { name: "Pifi", age: 18, gender: "female" ,grade: "A" }, { name: "Vadim", age: 31, gender: "male", grade: "B" }, { name: "Debra", age: 25, gender: "female", grade: "C" } ]) 
```
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("652920550671f21914043799"),
    '1': ObjectId("652920550671f2191404379a"),
    '2': ObjectId("652920550671f2191404379b"),
    '3': ObjectId("652920550671f2191404379c"),
    '4': ObjectId("652920550671f2191404379d")
  }
}
```
Т.к. в первом созданном документе (Alice), не было ключа «gender» мы увидем его при выводе и не сможем применить поиск или сортировку по нему.

    db.getCollection('test_collection').find({}, {name:1, gender:1})
```
[
  { _id: ObjectId("65291c360671f21914043798"), name: 'Alice' },
  {
    _id: ObjectId("652920550671f21914043799"),
    name: 'Boris',
    gender: 'male'
  },
  {
    _id: ObjectId("652920550671f2191404379a"),
    name: 'Alex',
    gender: 'male'
  },
  {
    _id: ObjectId("652920550671f2191404379b"),
    name: 'Pifi',
    gender: 'female'
  },
  {
    _id: ObjectId("652920550671f2191404379c"),
    name: 'Vadim',
    gender: 'male'
  },
  {
    _id: ObjectId("652920550671f2191404379d"),
    name: 'Debra',
    gender: 'female'
  }
]
```
### Обновление документов
Необходимо обновить и добавить «gender» в документ «Alice»
    
    db.test_collection.updateOne({name:"Alice"},{$set:{"gender": "female"}})
```
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```    
    db.getCollection('test_collection').find({name:"Alice"}, {name:1, gender:1})
```
[
  {
    _id: ObjectId("65291c360671f21914043798"),
    name: 'Alice',
    gender: 'female'
  }
]

### Поиск и сортировка
Сделаем поиск по ключу «gender» и с сортировкой по «age»
 
    db.getCollection('test_collection').find({gender:"female"}).sort({ age: 1 })
[
  {
    _id: ObjectId("652920550671f2191404379b"),
    name: 'Pifi',
    age: 18,
    gender: 'female',
    grade: 'A'
  },
  {
    _id: ObjectId("65291c360671f21914043798"),
    name: 'Alice',
    age: 25,
    grade: 'A',
    gender: 'female'
  },
  {
    _id: ObjectId("652920550671f2191404379d"),
    name: 'Debra',
    age: 25,
    gender: 'female',
    grade: 'C'
  }
]
```
Выполним поиск по нескольким условиям «age» - старше 24 и «grade» = A
    
    db.getCollection('test_collection').find({ $and: [{ age: { $gte: 24 } }, { 'grade': "A"} ]})
```
[
  {
    _id: ObjectId("65291c360671f21914043798"),
    name: 'Alice',
    age: 25,
    grade: 'A',
    gender: 'female'
  }
]
```
Результат всего один, по этому изменим «grade» с A на B

    db.getCollection('test_collection').find({ $and: [{ age: { $gte: 24 } }, { 'grade': "B"} ]})
```
[
  {
    _id: ObjectId("652920550671f21914043799"),
    name: 'Boris',
    age: 29,
    gender: 'male',
    grade: 'B'
  },
  {
    _id: ObjectId("652920550671f2191404379c"),
    name: 'Vadim',
    age: 31,
    gender: 'male',
    grade: 'B'
  }
]
```
### Работа с дампами баз и колекций
> *Для работы с дампами, необходима установка дополнительных инструментов для работы с mongoDB [here](https://www.mongodb.com/docs/database-tools/installation/installation/)*

    brew tap mongodb/brew
Установка инструментов для работы с mongoDB

    brew install mongodb-database-tools
Выполним импорт задампленных коллекций с документами используя bash скрипт [here](https://github.com/neelabalan/mongodb-sample-dataset)

```
#!/bin/bash
# vim:sw=4:ts=4:et:ai:ci:sr:nu:syntax=sh
##############################################################
# Usage ( * = optional ):                                    #
# ./script.sh *<db-address> *<db-port> *<username> *<password> #
##############################################################

if [ ! -z "$3" ]; then
    if [ ! -z "$4" ]; then
        echo "Using password authentication!"
        auth="--authenticationDatabase admin -u $3 -p $4"
    fi
fi

HOST=${1:-localhost} # default server is the localhost
PORT=${2:-27017}     # default port for MongoDB is 27017

for directory in *; do
    if [ -d "${directory}" ] ; then
        echo "$directory"
        for data_file in $directory/*; do
            mongoimport --drop --host $HOST --port $PORT --db "$directory" --collection "$(basename $data_file .json)" --file $data_file $auth
        done
    fi
done
```

    mongodb-sample-dataset-main % sh script.sh 
```
sample_airbnb
2023-10-13T16:27:15.073+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:15.074+0300	dropping: sample_airbnb.listingsAndReviews
2023-10-13T16:27:16.622+0300	5555 document(s) imported successfully. 0 document(s) failed to import.
sample_analytics
2023-10-13T16:27:16.646+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:16.647+0300	dropping: sample_analytics.accounts
2023-10-13T16:27:16.666+0300	1746 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:16.682+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:16.682+0300	dropping: sample_analytics.customers
2023-10-13T16:27:16.696+0300	500 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:16.711+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:16.712+0300	dropping: sample_analytics.transactions
2023-10-13T16:27:17.032+0300	1746 document(s) imported successfully. 0 document(s) failed to import.
sample_geospatial
2023-10-13T16:27:17.048+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:17.048+0300	dropping: sample_geospatial.shipwrecks
2023-10-13T16:27:17.198+0300	11095 document(s) imported successfully. 0 document(s) failed to import.
sample_mflix
2023-10-13T16:27:17.215+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:17.215+0300	dropping: sample_mflix.comments
2023-10-13T16:27:17.786+0300	50304 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:17.804+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:17.804+0300	dropping: sample_mflix.movies
2023-10-13T16:27:18.884+0300	23539 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:18.900+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:18.901+0300	dropping: sample_mflix.sessions
2023-10-13T16:27:18.907+0300	1 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:18.923+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:18.923+0300	dropping: sample_mflix.theaters
2023-10-13T16:27:18.947+0300	1564 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:18.964+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:18.964+0300	dropping: sample_mflix.users
2023-10-13T16:27:18.972+0300	185 document(s) imported successfully. 0 document(s) failed to import.
sample_supplies
2023-10-13T16:27:18.990+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:18.990+0300	dropping: sample_supplies.sales
2023-10-13T16:27:19.148+0300	5000 document(s) imported successfully. 0 document(s) failed to import.
sample_training
2023-10-13T16:27:19.163+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:19.164+0300	dropping: sample_training.companies
2023-10-13T16:27:20.047+0300	9500 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:20.065+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:20.065+0300	dropping: sample_training.grades
2023-10-13T16:27:21.420+0300	100000 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:21.439+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:21.440+0300	dropping: sample_training.inspections
2023-10-13T16:27:22.548+0300	80047 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:22.568+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:22.569+0300	dropping: sample_training.posts
2023-10-13T16:27:22.754+0300	500 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:22.778+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:22.778+0300	dropping: sample_training.routes
2023-10-13T16:27:23.423+0300	66985 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:23.438+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:23.439+0300	dropping: sample_training.stories
2023-10-13T16:27:23.792+0300	9842 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:23.812+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:23.812+0300	dropping: sample_training.trips
2023-10-13T16:27:24.013+0300	10000 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:24.029+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:24.029+0300	dropping: sample_training.tweets
2023-10-13T16:27:25.088+0300	24832 document(s) imported successfully. 0 document(s) failed to import.
2023-10-13T16:27:25.105+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:25.105+0300	dropping: sample_training.zips
2023-10-13T16:27:25.404+0300	29470 document(s) imported successfully. 0 document(s) failed to import.
sample_weatherdata
2023-10-13T16:27:25.423+0300	connected to: mongodb://localhost:27017/
2023-10-13T16:27:25.424+0300	dropping: sample_weatherdata.data
2023-10-13T16:27:26.051+0300	10000 document(s) imported successfully. 0 document(s) failed to import.
```
Проверяем, что базы появились
        
    show dbs
```
admin                40.00 KiB
config               72.00 KiB
local                40.00 KiB
sample_airbnb        52.18 MiB
sample_analytics      9.45 MiB
sample_geospatial   792.00 KiB
sample_mflix         28.70 MiB
sample_supplies     968.00 KiB
sample_training      60.91 MiB
sample_weatherdata    2.57 MiB
testBD               72.00 KiB
```
Теперь выполним дамп нашей созданой ранее базы "testBD"

    mongodump --forceTableScan --host localhost --port 27017  --archive='dump_testBD' --db='testBD'
```
2023-10-13T17:00:14.217+0300	writing testBD.test_collection to archive 'dump_testBD'
2023-10-13T17:00:14.223+0300	done dumping testBD.test_collection (6 documents)
```
И восстановим данные в новую базу "students"

    mongorestore --host localhost --port 27017 --archive='dump_testBD' --nsFrom='testBD.*' --nsTo='students.*' 
```
2023-10-13T17:00:32.567+0300	preparing collections to restore from
2023-10-13T17:00:32.571+0300	reading metadata for students.test_collection from archive 'dump_testBD'
2023-10-13T17:00:32.593+0300	restoring students.test_collection from archive 'dump_testBD'
2023-10-13T17:00:32.605+0300	finished restoring students.test_collection (6 documents, 0 failures)
2023-10-13T17:00:32.605+0300	no indexes to restore for collection students.test_collection
2023-10-13T17:00:32.605+0300	6 document(s) restored successfully. 0 document(s) failed to restore.
```

Убедимся, что у мы успешно восстановили базу
```
test> show dbs
admin                40.00 KiB
config              108.00 KiB
local                40.00 KiB
sample_airbnb        52.18 MiB
sample_analytics      9.45 MiB
sample_geospatial   792.00 KiB
sample_mflix         28.70 MiB
sample_supplies     968.00 KiB
sample_training      60.91 MiB
sample_weatherdata    2.57 MiB
students             40.00 KiB
testBD               72.00 KiB
test> use students
switched to db students
students> show collections
test_collection
students> db.getCollection('test_collection').find({ $and: [{ age: { $gte: 24 } }, { 'grade': "B"} ]})
[
  {
    _id: ObjectId("652920550671f21914043799"),
    name: 'Boris',
    age: 29,
    gender: 'male',
    grade: 'B'
  },
  {
    _id: ObjectId("652920550671f2191404379c"),
    name: 'Vadim',
    age: 31,
    gender: 'male',
    grade: 'B'
  }
]
```
