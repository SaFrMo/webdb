# WebDB

A database which reads and writes records on dat:// websites. [How it works](#how-it-works)

#### Example

Instantiate:

```js
// in the browser
const WebDB = require('beaker-webdb')
var webdb = new WebDB()

// in nodejs
const DatArchive = require('node-dat-archive')
const WebDB = require('beaker-webdb')
var webdb = new WebDB('./webdb', {DatArchive})
```

Define your schema:

```js
webdb.define('people', {
  // the schema that objects must match
  // (uses JSONSchema v6)
  schema: {
    type: 'object',
    properties: {
      firstName: {
        type: 'string'
      },
      lastName: {
        type: 'string'
      },
      age: {
        description: 'Age in years',
        type: 'integer',
        minimum: 0
      }
    },
    required: ['firstName', 'lastName']
  },

  // secondary indexes for fast queries (optional)
  index: ['lastName', 'lastName+firstName', 'age']
})
```

Then open the DB:

```js
await webdb.open()
```

Next we add source archives to be indexed. The source archives are persisted in IndexedDB/LevelDB, so this doesn't have to be done every run.

```js
await webdb.people.addSource(alicesUrl, {
  filePattern: '/people/*.json'
})
await webdb.people.addSource(bobsUrl, {
  filePattern: '/person.json'
})
await webdb.people.addSource(carlasUrl, {
  filePattern: [
    '/person.json',
    '/people/*.json'
  ]
})
```

Now we can begin querying the database for records.

```js
// get any person record where lastName === 'Roberts'
var mrRoberts = await webdb.people.get('lastName', 'Roberts')

// get any person record named Bob Roberts
var mrRoberts = await webdb.people.get('lastName+firstName', ['Roberts', 'Bob'])

// get all person records with the 'Roberts' lastname
var robertsFamily = await webdb.people
  .where('lastName')
  .equalsIgnoreCase('roberts')
  .toArray()

// get all person records with the 'Roberts' lastname
// and a firstname that starts with 'B'
// - this uses a compound index
var robertsFamilyWithaBName = await webdb.broadcasts
  .where('lastName+firstName')
  .between(['Roberts', 'b'], ['Roberts', 'b\uffff'])
  .toArray()

// get all person records on a given origin
// - origin is an auto-generated attribute
var personsOnBobsSite = await webdb.people
  .where('origin')
  .equals(bobsSiteUrl)
  .toArray()

// get the 30 oldest people indexed
var oldestPeople = await webdb.people
  .orderBy('age')
  .reverse() // oldest first
  .limit(30)
  .toArray()

// count the # of young people
var oldestPeople = await webdb.people
  .where('age')
  .belowOrEqual(18)
  .count()
```

We can also use WebDB to create, modify, and delete records (and their matching files).

```js
// set the record
await webdb.people.put(bobsUrl + '/person.json', {
  firstName: 'Bob',
  lastName: 'Roberts',
  age: 31
})

// update the record if it exists
await webdb.people.update(bobsUrl + '/person.json', {
  age: 32
})

// update or create the record
await webdb.people.upsert(bobsUrl + '/person.json', {
  age: 32
})

// delete the record
await webdb.people.delete(bobsUrl + '/person.json')

// update the spelling of all Roberts records
await webdb.people
  .where('lastName')
  .equals('Roberts')
  .update({lastName: 'Robertos'})

// increment the age of all people under 18
var oldestPeople = await webdb.people
  .where('age')
  .belowOrEqual(18)
  .update(record => {
    record.age = record.age + 1
  })

// delete the 30 oldest people
var oldestPeople = await webdb.people
  .orderBy('age')
  .reverse() // oldest first
  .limit(30)
  .delete()
```

## TODOs

WebDB is still in development.

 - [ ] More efficient key queries (currently loads full record from disk - could just load the keys)
 - [ ] Support for .or() queries

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Class: WebDB](#class-webdb)
  - [new WebDB([name])](#new-webdbname)
  - [WebDB.delete([name])](#webdbdeletename)
- [Instance: WebDB](#instance-webdb)
  - [webdb.open()](#webdbopen)
  - [webdb.close()](#webdbclose)
  - [webdb.define(name, definition)](#webdbdefinename-definition)
  - [webdb.addSource(url[, options])](#webdbaddsourceurl-options)
  - [webdb.removeSource(url)](#webdbremovesourceurl)
  - [webdb.listSources()](#webdblistsources)
  - [Event: 'open'](#event-open)
  - [Event: 'open-failed'](#event-open-failed)
  - [Event: 'versionchange'](#event-versionchange)
  - [Event: 'indexes-updated'](#event-indexes-updated)
- [Instance: WebDBTable](#instance-webdbtable)
  - [count()](#count)
  - [delete(url)](#deleteurl)
  - [each(fn)](#eachfn)
  - [filter(fn)](#filterfn)
  - [get(url)](#geturl)
  - [get(key, value)](#getkey-value)
  - [isRecordFile(url)](#isrecordfileurl)
  - [limit(n)](#limitn)
  - [listRecordFiles(url)](#listrecordfilesurl)
  - [name](#name)
  - [offset(n)](#offsetn)
  - [orderBy(key)](#orderbykey)
  - [put(url, record)](#puturl-record)
  - [query()](#query)
  - [reverse()](#reverse)
  - [schema](#schema)
  - [toArray()](#toarray)
  - [update(url, updates)](#updateurl-updates)
  - [update(url, fn)](#updateurl-fn)
  - [upsert(url, updates)](#upserturl-updates)
  - [where(key)](#wherekey)
  - [Event: 'index-updated'](#event-index-updated)
- [Instance: WebDBQuery](#instance-webdbquery)
  - [clone()](#clone)
  - [count()](#count-1)
  - [delete()](#delete)
  - [each(fn)](#eachfn-1)
  - [eachKey(fn)](#eachkeyfn)
  - [eachUrl(fn)](#eachurlfn)
  - [filter(fn)](#filterfn-1)
  - [first()](#first)
  - [keys()](#keys)
  - [last()](#last)
  - [limit(n)](#limitn-1)
  - [offset(n)](#offsetn-1)
  - [orderBy(key)](#orderbykey-1)
  - [put(record)](#putrecord)
  - [urls()](#urls)
  - [reverse()](#reverse-1)
  - [toArray()](#toarray-1)
  - [uniqueKeys()](#uniquekeys)
  - [until(fn)](#untilfn)
  - [update(updates)](#updateupdates)
  - [update(fn)](#updatefn)
  - [where(key)](#wherekey-1)
- [Instance: WebDBWhereClause](#instance-webdbwhereclause)
  - [above(value)](#abovevalue)
  - [aboveOrEqual(value)](#aboveorequalvalue)
  - [anyOf(values)](#anyofvalues)
  - [anyOfIgnoreCase(values)](#anyofignorecasevalues)
  - [below(value)](#belowvalue)
  - [belowOrEqual(value)](#beloworequalvalue)
  - [between(lowerValue, upperValue[, options])](#betweenlowervalue-uppervalue-options)
  - [equals(value)](#equalsvalue)
  - [equalsIgnoreCase(value)](#equalsignorecasevalue)
  - [noneOf(values)](#noneofvalues)
  - [notEqual(value)](#notequalvalue)
  - [startsWith(value)](#startswithvalue)
  - [startsWithAnyOf(values)](#startswithanyofvalues)
  - [startsWithAnyOfIgnoreCase(values)](#startswithanyofignorecasevalues)
  - [startsWithIgnoreCase(value)](#startswithignorecasevalue)
- [How it works](#how-it-works)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Class: WebDB

### new WebDB([name])

```js
var webdb = new WebDB('mydb')
```

 - `name` String. Defaults to `'webdb'`. If run in the browser, this will be the name of the IndexedDB instance. If run in NodeJS, this will be the path of the LevelDB folder.

### WebDB.delete([name])

```js
await WebDB.delete('mydb')
```

 - `name` String. Defaults to `'webdb'`. If run in the browser, this will be the name of the IndexedDB instance. If run in NodeJS, this will be the path of the LevelDB folder.
 - Returns Promise&lt;Void&gt;.

## Instance: WebDB

### webdb.open()

```js
await webdb.open()
```

 - Returns Promise&lt;Void&gt;.

### webdb.close()

```js
await webdb.close()
```

 - Returns Promise&lt;Void&gt;.

### webdb.define(name, definition)

 - `name` String. The name of the table.
 - `definition` Object.
   - `schema` Object. A [JSON Schema v6](http://json-schema.org/) definition.
   - `index` Array&lt;String&gt;. A list of attributes which should have secondary indexes produced for querying. Each `index` value is a keypath (see https://www.w3.org/TR/IndexedDB/#dfn-key-path).
 - Returns Void.

Create a new table on the `webdb` object.
The table will be set at `webdb.{name}` and be the `WebDBTable` type.

You can specify compound indexes with a `+` separator in the keypath.
You can also index each value of an array using the `*` sigil at the start of the name.
Some example index definitions:

```
one index               - index: 'firstName' 
two indexes             - index: ['firstName', 'lastName']
add a compound index    - index: ['firstName', 'lastName', 'firstName+lastName']
index an array's values - index: ['firstName', '*favoriteFruits']
```

Example:

```js
webdb.define('people', {
  // (uses JSONSchema v6)
  schema: {
    type: 'object',
    properties: {
      firstName: {
        type: 'string'
      },
      lastName: {
        type: 'string'
      },
      age: {
        description: 'Age in years',
        type: 'integer',
        minimum: 0
      }
    },
    required: ['firstName', 'lastName']
  },
  index: ['lastName', 'lastName+firstName', 'age']
})

await webdb.open()
// the new table will now be defined at webdb.people
```

### webdb.addSource(url[, options])

```js
await webdb.people.addSource('dat://foo.com', {
  filePattern: [
    '/myrecord.json',
    '/myrecords/*.json'
  ]
})
```

 - `url` String or DatArchive or Array&lt;String or DatArchive&gt;. The sites to index.
 - `options` Object.
   - `filePattern` String or Array&lt;String&gt;. An [anymatch](https://www.npmjs.com/package/anymatch) list of files to index.
 - Returns Promise&lt;Void&gt;.

### webdb.removeSource(url)

```js
await webdb.mytable.removeSource('dat://foo.com')
```

 - `url` String or DatArchive. The site to deindex.
 - Returns Promise&lt;Void&gt;.

### webdb.listSources()

```js
var urls = await webdb.mytable.listSources()
```

 - Returns Promise&lt;String&gt;.

### Event: 'open'

```js
webdb.on('open', () => {
  console.log('WebDB is ready for use')
})
```

### Event: 'open-failed'

```js
webdb.on('open-failed', (err) => {
  console.log('WebDB failed to open', err)
})
```

 - `error` Error.

### Event: 'versionchange'

```js
webdb.on('versionchange', () => {
  console.log('WebDB detected a change in schemas and rebuilt all data')
})
```

### Event: 'indexes-updated'

```js
webdb.on('indexes-updated', (url, version) => {
  console.log('Tables were updated for', url, 'at version', version)
})
```

 - `url` String. The site that was updated.
 - `version` Number. The version which was updated to.

## Instance: WebDBTable

### count()

```js
var numRecords = await webdb.mytable.count()
```

 - Returns Promise&lt;Number&gt;.

Count the number of records in the table.

### delete(url)

```js
await webdb.mytable.delete('dat://foo.com/bar.json')
```

 - Returns Promise&lt;Void&gt;.

Delete the record at the given URL.

### each(fn)

```js
await webdb.mytable.each(record => {
  console.log(record)
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Iterate over all records in the table with the given function.

### filter(fn)

```js
var records = await webdb.mytable.filter(record => {
  return (record.foo == 'bar')
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Boolean.
 - Returns WebDBQuery.

Start a new query and apply the given filter function to the resultset.

### get(url)

```js
var record = await webdb.mytable.get('dat://foo.com/myrecord.json')
```

 - `url` String. The URL of the record to fetch.
 - Returns Promise&lt;Object&gt;.

Get the record at the given URL.

### get(key, value)

```js
var record = await webdb.mytable.get('foo', 'bar')
```

 - `key` String. The keyname to search against.
 - `value` Any. The value to match against.
 - Promise&lt;Object&gt;.
 
Get the record first record to match the given key/value query.

### isRecordFile(url)

```js
var isRecord = webdb.mytable.isRecordFile('dat://foo.com/myrecord.json')
```

 - `url` String.
 - Returns Boolean.

### limit(n)

```js
var query = webdb.mytable.limit(10)
```

 - `n` Number.
 - Returns WebDBQuery.

### listRecordFiles(url)

```js
var recordFiles = await webdb.mytable.listRecordFiles('dat://foo.com')
```

 - `url` String.
 - Returns Promise&lt;Array&lt;Object&gt;&gt;. On each object:
   - `recordUrl` String.
   - `table` WebDBTable.

### name

 - String.

The name of the table.

### offset(n)

```js
var query = webdb.mytable.offset(5)
```

 - `n` Number.
 - Returns WebDBQuery.

### orderBy(key)

```js
var query = webdb.mytable.orderBy('foo')
```

 - `key` String.
 - Returns WebDBQuery.

### put(url, record)

```js
await webdb.mytable.put('dat://foo.com/myrecord.json', {foo: 'bar'})
```

 - `url` String.
 - `record` Object.
 - Returns Promise&lt;Void&gt;.

### query()

```js
var query = webdb.mytable.query()
```

 - Returns WebDBQuery.

### reverse()

```js
var query = webdb.mytable.reverse()
```

 - Returns WebDBQuery.

### schema

 - Object.

The schema definition for the table.

### toArray()

```js
var records = await webdb.mytable.toArray()
```

 - Returns Promise&lt;Array&gt;.

### update(url, updates)

```js
var wasUpdated = await webdb.mytable.update('dat://foo.com/myrecord.json', {foo: 'bar'})
```

 - `url` String. The record to update.
 - `updates` Object. The new values to set on the record.
 - Returns Promise&lt;Boolean&gt;.

### update(url, fn)

```js
var wasUpdated = await webdb.mytable.update('dat://foo.com/myrecord.json', record => {
  record.foo = 'bar'
  return record
})
```

 - `url` String. The record to update.
 - `fn` Function. A method to modify the record.
   - `record` Object. The record to modify.
   - Returns Object.
 - Returns Promise&lt;Boolean&gt;.

### upsert(url, updates)

```js
var didCreateNew = await webdb.mytable.upsert('dat://foo.com/myrecord.json', {foo: 'bar'})
```

 - `url` String. The record to update.
 - `updates` Object. The new values to set on the record.
 - Returns Promise&lt;Boolean&gt;.

### where(key)

```js
var whereClause = webdb.mytable.where('foo')
```

 - `key` String.
 - Returns IngestWhereClause.

### Event: 'index-updated'

```js
webdb.mytable.on('index-updated', (url, version) => {
  console.log('Table was updated for', url, 'at version', version)
})
```

 - `url` String. The site that was updated.
 - `version` Number. The version which was updated to.


## Instance: WebDBQuery

### clone()

```js
var query = webdb.mytable.query().clone()
```

 - Returns WebDBQuery.

### count()

```js
var numRecords = await webdb.mytable.query().count()
```

 - Returns Promise&lt;Number&gt;. The number of found records.

### delete()

```js
var numDeleted = await webdb.mytable.query().delete()
```

 - Returns Promise&lt;Number&gt;. The number of deleted records.

### each(fn)

```js
await webdb.mytable.query().each(record => {
  console.log(record)
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

### eachKey(fn)

```js
await webdb.mytable.query().eachKey(url => {
  console.log('URL =', url)
})
```

 - `fn` Function.
   - `key` String.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Gives the value of the query's primary key for each matching record.

The `key` is determined by the index being used.
By default, this is the `url` attribute, but it can be changed by using `where()` or `orderBy()`.

Example:

```js
await webdb.mytable.orderBy('age').eachKey(age => {
  console.log('Age =', age)
})
```

### eachUrl(fn)

```js
await webdb.mytable.query().eachUrl(url => {
  console.log('URL =', url)
})
```

 - `fn` Function.
   - `url` String.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Gives the URL of each matching record.

### filter(fn)

```js
var query = webdb.mytable.query().filter(record => {
  return record.foo == 'bar'
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Boolean.
 - Returns WebDBQuery.

### first()

```js
var record = await webdb.mytable.query().first()
```

 - Returns Promise&lt;Object&gt;.

### keys()

```js
var keys = await webdb.mytable.query().keys()
```

 - Returns Promise&lt;Array&lt;String&gt;&gt;.

The `key` is determined by the index being used.
By default, this is the `url` attribute, but it can be changed by using `where()` or `orderBy()`.

```js
var ages = await webdb.mytable.orderBy('age').keys()
```

### last()

```js
var record = await webdb.mytable.query().last()
```

 - Returns Promise&lt;Object&gt;.

### limit(n)

```js
var query = webdb.mytable.query().limit(10)
```

 - `n` Number.
 - Returns WebDBQuery.

### offset(n)

```js
var query = webdb.mytable.query().offset(10)
```

 - `n` Number.
 - Returns WebDBQuery.

### orderBy(key)

```js
var query = webdb.mytable.query().orderBy('foo')
```

 - `key` String.
 - Returns WebDBQuery.

### put(record)

```js
var numWritten = await webdb.mytable.query().put({foo: 'bar'})
```

 - `record` Object.
 - Returns Promise&lt;Number&gt;. The number of written records.

### urls()

```js
var urls = await webdb.mytable.query().urls()
```

 - Returns Promise&lt;Array&lt;String&gt;&gt;.

### reverse()

```js
var query = webdb.mytable.query().reverse()
```

 - Returns WebDBQuery.

### toArray()

```js
var records = await webdb.mytable.query().toArray()
```

 - Returns Promise&lt;Array&lt;Object&gt;&gt;.

### uniqueKeys()

```js
var keys = await webdb.mytable.query().uniqueKeys()
```

 - Returns Promise&lt;Array&lt;String&gt;&gt;.

The `key` is determined by the index being used.
By default, this is the `url` attribute, but it can be changed by using `where()` or `orderBy()`.

Example: 

```js
var ages = await webdb.mytable.orderBy('age').uniqueKeys()
```

### until(fn)

```js
var query = webdb.mytable.query().until(record => {
  return record.foo == 'bar'
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Boolean.
 - Returns WebDBQuery.

### update(updates)

```js
var numUpdated = await webdb.mytable.query().update({foo: 'bar'})
```

 - `updates` Object. The new values to set on the record.
 - Returns Promise&lt;Number&gt;. The number of updated records.

### update(fn)

```js
var numUpdated = await webdb.mytable.query().update(record => {
  record.foo = 'bar'
  return record
})
```

 - `fn` Function. A method to modify the record.
   - `record` Object. The record to modify.
   - Returns Object.
 - Returns Promise&lt;Number&gt;. The number of updated records.

### where(key)

```js
var whereClause = webdb.mytable.query().where('foo')
```

 - `key` String. The attribute to query against.
 - Returns IngestWhereClause.

## Instance: WebDBWhereClause

### above(value)

```js
var query = webdb.mytable.query().where('foo').above('bar')
var query = webdb.mytable.query().where('age').above(18)
```

 - `value` Any. The lower bound of the query.
 - Returns WebDBQuery.

### aboveOrEqual(value)

```js
var query = webdb.mytable.query().where('foo').aboveOrEqual('bar')
var query = webdb.mytable.query().where('age').aboveOrEqual(18)
```

 - `value` Any. The lower bound of the query.
 - Returns WebDBQuery.

### anyOf(values)

```js
var query = webdb.mytable.query().where('foo').anyOf(['bar', 'baz'])
```

 - `values` Array&lt;Any&gt;.
 - Returns WebDBQuery.

### anyOfIgnoreCase(values)

```js
var query = webdb.mytable.query().where('foo').anyOfIgnoreCase(['bar', 'baz'])
```

 - `values` Array&lt;Any&gt;.
 - Returns WebDBQuery.

### below(value)

```js
var query = webdb.mytable.query().where('foo').below('bar')
var query = webdb.mytable.query().where('age').below(18)
```

 - `value` Any. The upper bound of the query.
 - Returns WebDBQuery.

### belowOrEqual(value)

```js
var query = webdb.mytable.query().where('foo').belowOrEqual('bar')
var query = webdb.mytable.query().where('age').belowOrEqual(18)
```

 - `value` Any. The upper bound of the query.
 - Returns WebDBQuery.

### between(lowerValue, upperValue[, options])

```js
var query = webdb.mytable.query().where('foo').between('bar', 'baz', {includeUpper: true, includeLower: true})
var query = webdb.mytable.query().where('age').between(18, 55, {includeLower: true})
```

 - `lowerValue` Any.
 - `upperValue` Any.
 - `options` Object.
   - `includeUpper` Boolean.
   - `includeLower` Boolean.
 - Returns WebDBQuery.

### equals(value)

```js
var query = webdb.mytable.query().where('foo').equals('bar')
```

 - `value` Any.
 - Returns WebDBQuery.

### equalsIgnoreCase(value)

```js
var query = webdb.mytable.query().where('foo').equalsIgnoreCase('bar')
```

 - `value` Any.
 - Returns WebDBQuery.

### noneOf(values)

```js
var query = webdb.mytable.query().where('foo').noneOf(['bar', 'baz'])
```

 - `values` Array&lt;Any&gt;.
 - Returns WebDBQuery.

### notEqual(value)

```js
var query = webdb.mytable.query().where('foo').notEqual('bar')
```

 - `value` Any.
 - Returns WebDBQuery.

### startsWith(value)

```js
var query = webdb.mytable.query().where('foo').startsWith('ba')
```

 - `value` Any.
 - Returns WebDBQuery.

### startsWithAnyOf(values)

```js
var query = webdb.mytable.query().where('foo').startsWithAnyOf(['ba', 'bu'])
```

 - `values` Array&lt;Any&gt;.
 - Returns WebDBQuery.


### startsWithAnyOfIgnoreCase(values)

```js
var query = webdb.mytable.query().where('foo').startsWithAnyOfIgnoreCase(['ba', 'bu'])
```

 - `values` Array&lt;Any&gt;.
 - Returns WebDBQuery.


### startsWithIgnoreCase(value)

```js
var query = webdb.mytable.query().where('foo').startsWithIgnoreCase('ba')
```

 - `value` Any.
 - Returns WebDBQuery.



## How it works

WebDB abstracts over the [DatArchive API](https://beakerbrowser.com/docs/apis/dat.html) to provide a simple database-like interface. It is inspired by [Dexie.js](https://github.com/dfahlander/Dexie.js) and built using LevelDB. (In the browser, it runs on IndexedDB using [level.js](https://github.com/maxogden/level.js).

WebDB works by scanning a set of source archives for files that match a path pattern. Those files are indexed ("ingested") so that they can be queried easily. WebDB also provides a simple interface for adding, editing, and removing records on the archives that the local user owns.

WebDB sits on top of Dat archives. It duplicates the data it's handling into IndexedDB, and that duplicated data acts as a throwaway cache -- it can be reconstructed at any time from the Dat archives.

WebDB treats individual files in the Dat archive as individual records in a table. As a result, there's a direct mapping for each table to a folder of .json files. For instance, if you had a 'tweets' table, it would map to the `/tweets/*.json` files. WebDB's mutators, such as put or add or update, simply write those json files. WebDB's readers & query-ers, such as get() or where(), read from the IndexedDB cache.

WebDB watches its source archives for changes to the json files. When they change, it reads them and updates IndexedDB, thus the query results stay up-to-date. The flow is, roughly: `put() -&gt; archive/tweets/12345.json -&gt; indexer -&gt; indexeddb -&gt; get()`.