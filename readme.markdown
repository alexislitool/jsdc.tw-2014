# micro database (小资料库)

substack
  
  http://substack.net
  https://{github,twitter}.com/substack
  https://npmjs.org/~substack

---

# micro database (小资料库)

原谅坏的翻译!

---

# unix philosophy

unix学说:

```
'Make each program do one thing well.'
```

---

# unix philosophy

unix学说:

```
'Make each program do one thing well.'
+ 'To do a new job, build afresh rather than complicate'
+ 'old programs by adding new features.'
```

---

# unix philosophy

* modularity (模組) write simple parts connected by clean interfaces
* composition (作文) design programs to be connected with other programs
* parsimony (吝啬) write a big program only as a last resort

---

# leveldb

tiny embedded database

内含的小资料库

---

# leveldb

鍵值儲存

---

# leveldb

* db.get() - 取
* db.put() - 插入物
* db.del() - 删除
* db.batch() - 原子

---

# leveldb

* db.get() - 取
* db.put() - 插入物
* db.del() - 删除
* db.batch() - 原子
* db.createReadStream()

---

# leveldb

* db.get() - 取
* db.put() - 插入物
* db.del() - 删除
* db.batch() - 原子
* db.createReadStream({ gt: ..., lt: ... })

---

# lexicographic ordering

字典序

---

# lexicographic ordering

字典序:

```
aaa
bbb
ccc
```

---

# lexicographic ordering

字典序:

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

字典序 with bytewise:

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

字典序 with bytewise:

```
null
Number
String
Array (componentwise)
Object (componentwise string-keyed key/value pairs)
undefined
```

---

# bytewise

字典序 with bytewise:

```
['user','dominictarr']
['user','mafintosh']
['user','substack']
```

---

# bytewise

字典序 with bytewise:

```
['user','dominictarr']
['user','mafintosh']
['user','substack']
['post',1412991992967,'substack']
['post',1412992015628,'dominictarr']
['post',1412992057388,'substack']
['post',1412992068543,'mafintosh']
```

---

# bytewise

字典序 with bytewise:

```
['user','dominictarr']
['user','mafintosh']
['user','substack']
['post',1412991992967,'substack']
['post',1412992015628,'dominictarr']
['post',1412992057388,'substack']
['post',1412992068543,'mafintosh']
['post-user','dominictarr',1412992015628]
['post-user','mafintosh',1412992068543]
['post-user','substack',1412991992967]
['post-user','substack',1412992057388]
```

---

# sublevel

a database inside a database!

---

# sublevel

nested keys

and nested encodings!

---

# translation database 翻译资料库

parse-dictd

---

# translation 翻译资料库

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
