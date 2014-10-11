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

# translation 翻译资料库 (constructor)

``` js



module.exports = Translate;

function Translate (db) {
    if (!(this instanceof Translate)) return new Translate(db);
}
```

---

# translation 翻译资料库 (constructor)

``` js
var sublevel = require('level-sublevel/bytewise');
var bytewise = require('bytewise');

module.exports = Translate;

function Translate (db) {
    if (!(this instanceof Translate)) return new Translate(db);
    this.db = sublevel(db, {
        keyEncoding: bytewise,
        valueEncoding: 'json'
    });
}
```

---

# translation 翻译资料库 (link)

``` js
Translate.prototype.link = function (a, b, cb) {
    var rows = [
        // ...
    ];
    this.db.batch(rows, cb);
};
```

---

# translation 翻译资料库 (link)

``` js
Translate.prototype.link = function (a, b, cb) {
    var rows = [
        { key: [ 'link', a.lang, a.word, b.lang, b.word ], value: 0 }
        
    ];
    this.db.batch(rows, cb);
};
```

---

# translation 翻译资料库 (link)

``` js
Translate.prototype.link = function (a, b, cb) {
    var rows = [
        { key: [ 'link', a.lang, a.word, b.lang, b.word ], value: 0 },
        { key: [ 'link', b.lang, b.word, a.lang, a.word ], value: 0 }
    ];
    this.db.batch(rows, cb);
};
```

---

# translation 翻译资料库

``` js
Translate.prototype.get = function (opts) {
    var s = this.db.createReadStream({
        gt: [ 'link', opts.from, opts.word, opts.to, null ],
        lt: [ 'link', opts.from, opts.word, opts.to, undefined ]
    });
    var r = through.obj(function (row, enc, next) {
        this.push({ lang: row.key[3], word: row.key[4] });
        next();
    });
    s.on('error', r.emit.bind(r, 'error'));
    return s.pipe(r);
};
```
---

# translation (import)

``` js


var fs = require('fs');



var dstream = fs.createReadStream(process.argv[2]);
var istream = fs.createReadStream(process.argv[3]);
```

---

# translation (import)

``` js


var fs = require('fs');
var gunzip = require('zlib').createGunzip;

var dstream = fs.createReadStream(process.argv[2]).pipe(gunzip());
var istream = fs.createReadStream(process.argv[3]);
```

---

# translation (import)

``` js
var parse = require('parse-dictd');
var through = require('through2');
var fs = require('fs');
var gunzip = require('zlib').createGunzip;

var dstream = fs.createReadStream(process.argv[2]).pipe(gunzip());
var istream = fs.createReadStream(process.argv[3]);

parse(dstream, istream).pipe(through.obj(function (row, enc, next) {
    // ...
}));
```

---

# translation (import)

``` js
var minimist = require('minimist');
var argv = minimist(process.argv.slice(2));

// ...

parse(dstream, istream).pipe(through.obj(function (row, enc, next) {
    var a = { word: row.from, lang: argv.from };
    var b = { word: row.to[0], lang: argv.to };
    ddb.link(a, b, next);
}));
```

---

# translation (import)

``` js
var parse = require('parse-dictd');
var through = require('through2');
var fs = require('fs');
var gunzip = require('zlib').createGunzip;
var minimist = require('minimist');

var ddb = require('dictdb')(require('level')('./dict.db'));

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

``` js
var ddb = require('dictdb')(require('level')('./dict.db'));
var argv = require('minimist')(process.argv.slice(2));



```

---

# translation (query)

``` js
var ddb = require('dictdb')(require('level')('./dict.db'));
var argv = require('minimist')(process.argv.slice(2));

var q = { word: argv._.join(' '), from: argv.from, to: argv.to };

```

---

# translation (query)

``` js
var ddb = require('dictdb')(require('level')('./dict.db'));
var argv = require('minimist')(process.argv.slice(2));

var q = { word: argv._.join(' '), from: argv.from, to: argv.to };
ddb.get(q).on('data', console.log);
```

---

# translation demo!

演示!

---

# accountdown

modular user accounts!

---

# accountdown

```

var db = require('level')('./users.db')
```

---

# accountdown

```
var accountdown = require('accountdown')
var db = require('level')('./users.db')

var users = accountdown(db, {
    // ...
});
```

---

# accountdown

``` js
var accountdown = require('accountdown')
var db = require('level')('./users.db')

var users = accountdown(db, {
    login: { basic: require('accountdown-basic') },
    // ...
});
```

---

# accountdown

``` js
var accountdown = require('accountdown')
var db = require('level')('./users.db')

var users = accountdown(db, {
    login: { basic: require('accountdown-basic') },
    rfid: { basic: require('accountdown-rfid') },
    gpg: { basic: require('accountdown-gpg') }
    // ...or whatever!
});
```

---

# accountdown

``` js
// ...

users.create('substack', {
    login: { basic: { username: 'substack', password: 'beepboop' } },
    value: { bio: 'oh hello' }
});
```

---

# accountdown

```
// ...

users.list().on('data', console.log)
```

---

# accountdown

```
// ...

users.verify('basic', creds, function (err, ok, id) {
    if (err) console.error(err);
    console.log('ok=', ok);
    console.log('id=', id);
})
```

---

# level-party

open a leveldb directory from multiple processes

---

# accountdown-command

manage accountdown accounts on the command-line

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
```

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
var command = require('accountdown-command');
```

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
var command = require('accountdown-command');

if (process.argv[2] === 'server') {
    var http = require('http');
    var through = require('through2');
    http.createServer(function (req, res) {
        users.list().pipe(through.obj(function (row, enc, next) {
            this.push(row.key + '\n');
            next();
        })).pipe(res);
    }).listen(5000);
}
```

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
var command = require('accountdown-command');

if (process.argv[2] === 'server') {
    require('./handle.js')(users).listen(5000);
}
```

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
var command = require('accountdown-command');

if (process.argv[2] === 'users') {
    // ...
}
else if (process.argv[2] === 'server') {
    require('./handle.js')(users).listen(5000);
}
```

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
var command = require('accountdown-command');

var users = accountdown(db, {
    login: { basic: require('accountdown-basic') }
});

if (process.argv[2] === 'users') {
    command(users, process.argv.slice(3), function (err) {
        // ...
        
    }).pipe(process.stdout);
}
else if (process.argv[2] === 'server') {
    require('./handle.js')(users).listen(5000);
}
```

---

# accountdown-command

``` js
var accountdown = require('accountdown');
var db = require('level-party')('./accounts.db');
var command = require('accountdown-command');

var users = accountdown(db, {
    login: { basic: require('accountdown-basic') }
});

if (process.argv[2] === 'users') {
    command(users, process.argv.slice(3), function (err) {
        if (err) console.error(err);
        db.close();
    }).pipe(process.stdout);
}
else if (process.argv[2] === 'server') {
    require('./handle.js')(users).listen(5000);
}
```

---

# accountdown-command demo!

演示!

---

# content-addressable-blob-store

俳句:

```
Key of document
is the hash of its content.
Addressable blob.
```

---

# content-addressable-blob-store

```
解答公文杂乱信号
可设定位址的一滴
```

---

# content-addressable-blob-store demo!

演示!

---

# replication is easy!

when you use content-addressable data!

---

# hash-exchange

exchange cryptographic hashes of content
for multi-master replication

---

# hash-exchange

``` js
var shasum = require('shasum');

var messages = process.argv.slice(2);
var data = {};
messages.forEach(function (msg) { data[shasum(msg)] = msg });
```

---

# hash-exchange

``` js
// ...

var exchange = require('hash-exchange');

var ex = exchange(function (hash) {
    // ...
});
```

---

# hash-exchange

``` js
// ...
var through = require('through2');
var exchange = require('hash-exchange');

var ex = exchange(function (hash) {
    var r = through();
    r.end(data[hash]);
    return r;
});
```

---

# hash-exchange

``` js
// ...

ex.provide(Object.keys(data));
```

---

# hash-exchange

``` js
// ...

ex.on('available', function (hashes) {
    ex.request(hashes);
});
```

---

# hash-exchange

``` js
var concat = require('concat-stream');
// ...

ex.on('response', function (hash, stream) {
    stream.pipe(concat(function (body) {
        console.error('# BEGIN ' + hash);
        console.error(body.toString('utf8'));
        console.error('# END ' + hash);
    }));
});
```

---

# hash-exchange

``` js
// ...

process.stdin.pipe(ex).pipe(process.stdout);
```

---

# hash-exchange demo!

演示!

---

# forkdb

forkdable, offline-first data store

---

# forkdb

forkdable, offline-first data store
with multi-master replication

---

# forkdb

forkdable, offline-first data store
with multi-master replication
using leveldb

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

# forkdb demo!

演示!

---

# wikidb

wiki database on top of forkdb!

```
var inherits = require('inherits');
inherits(WikiDB, ForkDB);
```

# wikidb demo!

演示!

---

# more micro databases built on leveldb

* maildb - database for imap and smtp, used by eelmail
* batchdb - job queue database
* treedb - database for trees

---

# learn more

* levelmeup on http://nodeschool.io
* [
    'level', 'dictdb','accountdown','forkdb','wikidb','treedb',
    'content-addressable-blob-store','hash-exchange','shasum',
  ].forEach(function (x) {
    console.log('https://npmjs.org/package/' + x)
  })
* coming soon: wikidb-powered code cookbook

---

# thanks

```
$ node get.js --from en --to zh thank you
谢谢你
```

---

# thanks

```
$ node get.js --from en --to zh thank you
谢谢你
$ node get.js --from en --to zh goodbye
再会
再见
```
