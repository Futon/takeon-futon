# JSONStream

streaming JSON.parse and stringify. Don't wait for all the results to finish before doing stuff.

## Jam install

```
jam install JSONStream
```


## example

```javascript

            require(['JSONStream'], function (JSONStream) {

                var xhr = new XMLHttpRequest()
                xhr.open("GET", "http://proxy.max.iriscouch.com:1234/oakland_assessor/_all_docs?include_docs=true", true);
                var stream = new JSONStream.XHRStream(xhr)


                var json = JSONStream.parse(['rows', /./, 'doc'])
                stream.pipe(json)
                json.on('data', function(doc) {
                   console.log(doc);
                });


            });
```

Couchdb does not yet allow CORS requests, so the above example is using a proxy in front of the actual couch. See [CORS Proxy](https://github.com/daleharvey/CORS-Proxy).



## JSONStream.parse(path)

usally, a json API will return a list of objects.  

`path` should be an array of property names and/or `RedExp`s.  
any object that matches the path will be emitted as 'data' (and `pipe`d down stream)

if `path` is empty or null, or if no matches are made:  
JSONStream.parse will only one 'data': the root object.

(this is useful when there is an error, because the error will probably not match your path)

### example

query a couchdb view:

``` bash
curl -sS localhost:5984/tests/_all_docs&include_docs=true
```
you will get something like this:

``` js
{"total_rows":129,"offset":0,"rows":[
  { "id":"change1_0.6995461115147918"
  , "key":"change1_0.6995461115147918"
  , "value":{"rev":"1-e240bae28c7bb3667f02760f6398d508"}
  , "doc":{
      "_id":  "change1_0.6995461115147918"
    , "_rev": "1-e240bae28c7bb3667f02760f6398d508","hello":1}
  },
  { "id":"change2_0.6995461115147918"
  , "key":"change2_0.6995461115147918"
  , "value":{"rev":"1-13677d36b98c0c075145bb8975105153"}
  , "doc":{
      "_id":"change2_0.6995461115147918"
    , "_rev":"1-13677d36b98c0c075145bb8975105153"
    , "hello":2
    }
  },
]}

```

we are probably most interested in the `rows.*.docs`  

create a `Stream` that parses the documents from the feed like this:

``` js
JSONStream.parse(['rows', /./, 'doc']) //rows, ANYTHING, doc
``` 
awesome!

## JSONStream.stringify(open, sep, close)

Create a writable stream.

you may pass in custom `open`, `close`, and `seperator` strings.  
But, by default, `JSONStream.stringify()` will create an array,  
(with default options `open='[\n', sep='\n,\n', close='\n]\n'`)

If you call `JSONStream.stringify(false)`   
the elements will only be seperated by a newline.  

If you only write one item this will be valid JSON.  

If you write many items,  
you can use a `RegExp` to split it into valid chunks.

## JSONStream.stringifyObject(open, sep, close)

Very much like `JSONStream.stringify`,
but creates a writable stream for objects instead of arrays.

Accordingly, `open='{\n', sep='\n,\n', close='\n}\n'`.

When you `.write()` to the stream you must supply an array with `[ key, data ]`
as the first argument.

## numbers

There are occasional problems parsing and unparsing very precise numbers.  

I have opened an issue here:

https://github.com/creationix/jsonparse/issues/2

+1

## Acknowlegements

Max Ogden!!!
this module depends on https://github.com/creationix/jsonparse  
by Tim Caswell  
and also thanks to Florent Jaby for teaching me about parsing with:
https://github.com/Floby/node-json-streams
