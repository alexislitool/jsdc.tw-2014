# micro databases

---

# leveldb

tiny embedded database

---

# bytewise

lexicographic ordering

---

# bytewise

```
aaa
bbb
ccc
```

---

# bytewise

```
1
10
14
2
4
6
9
```

---

# bytewise

with bytewise:

```
1
2
4
6
9
10
14
```

---

# bytewise

also with bytewise:

```
null
Number
String
Array (componentwise)
Object (componentwise string-keyed key/value pairs)
undefined
```

---

# sublevel

a database inside a database!

---

# sublevel

nested keys

and nested encodings!

---

# translation database

parse-dictd

---

# translation (import)

``` js
var parse = require('parse-dictd');
var through = require('through2');
var fs = require('fs');
var gunzip = require('zlib').createGunzip;
var minimist = require('minimist');

var ddb = require('dictdb')(require('level')('/tmp/dict.db'));

var argv = minimist(process.argv.slice(2));
var dstream = fs.createReadStream(argv._[0]).pipe(gunzip());
var istream = fs.createReadStream(argv._[1]);

parse(dstream, istream).pipe(through.obj(function (row, enc, next) {
    var a = { word: row.from, lang: argv.from };
    var b = { word: row.to[0], lang: argv.to };
    ddb.link(a, b, next);
}));
```

---

# translation (query)

```
var minimist = require('minimist');
var ddb = require('dictdb')(require('level')('/tmp/dict.db'));

var argv = minimist(process.argv.slice(2));
var q = { word: argv._.join(' '), from: argv.from, to: argv.to };
ddb.get(q, function (err, results) {
    if (err) return console.error(err);
    results.forEach(function (r) { console.log(r.word) });
});
```

# accountdown

```
content-addressable
```

---

# level-party

---

# content-addressable-blob-store

---

# forkdb

content-addressable storage using leveldb

---

# forkdb

```
A
↓
B
↓
C
```

---

# forkdb

```








C  775dc2e743389c014d360a6da7f432b52a478dd6
   {}
```

---

# forkdb

```




B  b921c05a26a02ebbad1d7ef41a53f88d7f623a4e
   {"prev":"775dc2e743389c014d360a6da7f432b52a478dd6"}
↓

C  775dc2e743389c014d360a6da7f432b52a478dd6
   {}
```

---

# forkdb

```
A  d01f38b4e1f3502375192279503f8c9c9842a50d
   {"prev":"b921c05a26a02ebbad1d7ef41a53f88d7f623a4e"}
↓

B  b921c05a26a02ebbad1d7ef41a53f88d7f623a4e
   {"prev":"775dc2e743389c014d360a6da7f432b52a478dd6"}
↓

C  775dc2e743389c014d360a6da7f432b52a478dd6
   {}
```

---

# forkdb

```
A  X
↓  ⎥
B  ↲
↓
C
```

---

# replication is easy!

content-addressable storage

---

# replication is easy!

content-addressable storage

---

# wikidb

---

# maildb

---

# eelmail

---

# batchdb

---

# email server into batchdb

---

# CYBERWIZARD INSTITUTE
