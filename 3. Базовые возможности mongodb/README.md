# Домашнее задание по 3му уроку
*Примечание: Работы производились на macOS 14.0 c Docker Desktop 4.24.0 (122432)*

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
*Примечание: Если в базу не добавлена хотя бы одна коллекция, а в ней нет хотя бы одного документа, то база не будет создана, что видно при повторном вызове списка баз, после создания базы «testBD»*

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
