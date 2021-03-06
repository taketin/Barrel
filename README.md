Barrel
=================

A simple type-safe CoreData library for Swift.

##Installation
```ruby
platform :ios, "8.0"
use_frameworks!

pod 'Barrel', :git => 'https://github.com/tarunon/Barrel.git'
```

##Summary

Swift is a type-safe programing language and supported type-inference.

Before
```swift
func findPerson(age: Int) -> [Person] {
  let fetchRequest = NSFetchRequest(entityName: "Person")
  fetchRequest.predicate = NSPredicate(format: "age == %i", age)
  fetchRequest.sortDescriptor = [NSSortDescriptor(key: "name", ascending: true)]
  return context.executeFetchRequest(fetchRequest, error: nil) as? [Person] ?? []
}

func createPerson(name: String, age: Int) -> Person {
  let person = NSEntityDescription.insertNewObjectForEntityForName("Person", inManagedObjectContext: context) as! T
  person.name = name
  person.age = age
  return person
}
```

After
```swift
func findPerson(age: Int) -> [Person] {
  return context.fetch()
    .filter{ $0.age == age }
    .orderBy{ $0.name < $1.name }
    .execute().all()
}

func createPerson(name: String, age: Int) -> Person {
  return context.insert().setValues{
    $0.name = name
    $0.age = age
    }.insert()
}
```

##Fetch

Barrel provides fetch methods.

###Type-safe
Barrel's fetch method is type-safe.
```swift
let persons = context.fetch(Person).execute().all()
```
If the type is defined, fetch's argument can be omitted.
```swift
let persons: [Person] = context.fetch().execute().all()
```

###Enum value
A result of Barrel's fetch is Enum value Array or NSError.
```swift
let personsResult = context.fetch(Person).execute()
switch personsResult {
case .Succeed(let persons):
    // case of Array of Person
case .Failed(let error):
    // case of NSError
}
```

###Method chaining
Barrel is defined condition of fetch using method chaining.
```swift
let persons = context.fetch(Person)
  .filter(NSPredicate(format: "name == %@", "John"))
  .orderBy(NSSortDescriptor(key: "age", ascending: true))
  .execute().all()
```

###Closure
Barrel can be defined condition of fetch using closure.
```swift
let persons = context.fetch(Person)
  .filter{ $0.name == "John" }
  .orderBy{ $0.age < $1.age }
  .execute().all()
```


##Insert

Barrel provides insert methods.
Type-safe and Closure support.

```swift
let person = context.insert(Person).setValues{
  $0.name = "John"
  $0.age = 24
  }.insert()
```

And support getOrInsert method.

```swift
let person = context.insert(Person).setValues{
  $0.name = "John"
  $0.age = 24
  }.getOrInsert()
```


##Others

Barrel has more functions.

###Aggregate and Grouping

Support aggregate method,
```swift
let maxAge = context.fetch(Person)
  .aggregate{ $0.max($1.age) }
  .execute().get()!
// maxAge => ["maxAge": XX]
```

and grouping.
```swift
let maxAgePerName = context.fetch(Person)
  .aggregate{ $0.max($1.age) }
  .aggregate{ $1.name }
  .groupBy{ $1.name }
  .execute().all()
// maxAgePerName => [["maxAge" : XX, "name": "YY"], ...]
```

###ResultsController
NSFetchResultsController is not also type-safe.
Barrel supports Type-safe ResultsController object.
```swift
let resultsController: ResultsController<Person> = context.fetch(Person)
  .orderBy{ $0.age < $1.age }
  .resultsController(sectionKeyPath: nil, cacheName: nil)
let person = resultsController.objectAtIndexPath(NSIndexPath(forRow: 0, inSection: 0))
```
