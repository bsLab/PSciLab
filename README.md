# PSciLAB: Hybrid Distributed-Parallel Scientific Software Laboratory

## Author

```
PD Dr. Stefan Bosse
Institute for Digitisation and Sensor Data Science
28359 Bremen, Germany
```

## Cite


*S. Bosse, PSciLab: An Unified Distributed and Parallel Software Framework for Data Analysis, Simulation and Machine Learning—Design Practice, Software Architecture, and User Experience , Appl. Sci. 2022, 12(6), 2887; 10.3390/app12062887*


## Components

### WorkBook

The WorkBook provides a GUI for scientific computing using widely availabke Web browsers. the WorkBook is organised by snippets, mainly code, visualisation (Plots, tables, graphs= and Markdown text snippets, similar to Pyhon Jupier workbooks. In contrast to Python-based computing requiring Python software, the WOrkBook is programmed in JavaScript (JS) and directly executed by the Web browsers JS VM. 

There is a large set of plug-ins that can be loaded at run-time (`load("plugin")`):

- Extended *math.plugin* including extended vector and matrix modules supporting typed arrays
- Universial Machine Learning provided by the *ml.plugin*
- and many more...

### WorkShell

The terminal counterpart of the WorkBook without visualisation (except text tables). WorkBook code can typically directly executed by the WorkShell. The WorkShell contains already the math and ML plug-ins.

### Web WorkShell

The WorkShell for the Web browser providing only a shell terminal and script exeuction that can be configured by URL parameter. The WortkShell requires *node.js* (at least version 8) and some additional modules (*deasync*, *ws*, *ffi*, *ref*) that can be installed using the *npmÜ tool (`npm install deasync`). Note that there are some native code modules that require C++ compiler support.

## Parellel Processing

The WorkBook as well as the WOrkShell can spawn an aribitrary number of worker processes that can be executed in parallel. Worker processes are created by an unified Worker class object:

```javascript
var worker = await new Worker();
await worker.ready()
var result = await worker.eval(function (x) { return x*x },2);
print(result) // 4
worker.eval(async function () { var x=await receive(); send(x*x) });
worker.send(2)
print(result) // 4
worker.kill()
```

Workers provide a service loop that proecsses code and function request by the parent process.

## Distributed Computing

A WorkShell can provide a Web (HTTP/WS) service API that enables scripts to create and access workers on WorkShell remotely, basically from within the WorkBook.
To enable the WotkShell service API start the Workshell witgh the following options:

```
worksh -p 127.0.0.1:5104
```


In the WorkBook play the following code (the same from above!):

```javascript
var worker = await new Worker('ws:127.0.9.1:5104');
await worker.ready()
var result = await worker.eval(function (x) { return x*x },2);
print(result) // 4
worker.eval(async function () { var x=await receive(); send(x*x) });
worker.send(2)
print(result) // 4
worker.kill()
```

## Data Management

Data is stored and exchanged by worker processes using a SQLite data base (or multiple). The *sqld* server provides a RPC communication interface to create and access data bases using JSON queries.

A *sqld* server can be cold started by providing a directory where sql data bases can be stored:

```
sqld -d /home/user/data:9999
```

The *sqld* server requires the *better-sqlite@2.10* module that must be installed by *npm*.

A WorkBook or WorkShell script can create and access data bases the following way:

```javascript
load('math.plugin')
var db = DB.sqlA('localhost:9999');
print(await db.databases())
var stat = await db.createDB('mydb1');
// will be created in /home/user/data/mydb1.db
if (Utils.isError(stat)) print('failed');
var stat = await db.create('mytab1',{id:'int unique', name:'text', data:'blob'}); 
if (Utils.isError(stat)) print('failed');
db.insert('mytab1',{
  id:1,
  name:'data1',
  data:Math.MatrixTA.Random(4,3).data
})
print(await db.select('mytab1','id,name'))
```
