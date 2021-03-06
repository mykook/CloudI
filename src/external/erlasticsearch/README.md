ErlasticSearch
=========
A thrift based erlang client for [ElasticSearch](http://www.elasticsearch.org/).

It incorporates a connection-pool based on [poolboy](https://github.com/devinus/poolboy), which is by far the easiest and least-painful way to get going. It - basically - just works.

Alternately, you can start and stop individual (named/registered) _gen_servers_ and access ElasticSearch through them.



Installation
============
Add this as a rebar dependency to your project.

1. Be sure to set up ElasticSearch to support thrift!
   * Install the thrift plugin (available [here](https://github.com/elasticsearch/elasticsearch-transport-thrift))
      * Probably something like --> ```bin/plugin -install elasticsearch/elasticsearch-transport-thrift/1.5.0.```
   * You'll need to add (at least) the following settings to config.yaml
      * ```thrift.port: 9500```
      * ```thrift.protocol: 'binary'```
    * You might want to set the port to whatever you want instead of ```9500```.  Mind you, in that case you might need to update ```app.config``` and/or your ```connection_options``` in your application/erlasticsearch setup too.
    * Start ElasticSearch
       * If you plan on running the tests, you probably want to do this.
       * Heck, if you plan on using ElasticSearch, you probably want to do this.
       * If you plan on running the tests, you might want to run it in 'in-memory' mode.
          * Probably something like --> ```elasticsearch -f -Des.index.storage.type=memory -Des.config=/usr/local/opt/elasticsearch/config/elasticsearch.yml```
1. Update your environment with the following ```connection_options``` (look in [app.config](https://github.com/dieswaytoofast/erlasticsearch/blob/master/app.config) for examples)
   * ```thrift_options``` (default : *[{framed, true}]*)
   * ```thrift_host``` (default : *"localhost"*)
   * ```thrift_port``` (default : *9500*)
   * ```binary_response``` (default *true*. When *false*, this will run ```jsx:decode``` on the ES response, and send the tuples back to you, instead of one long binary)
   * Poolboy specific information
      * ```pools```
         * Note that if you are using the _connection pooled_ version (*which I recommend! Its easy! It works!*), you can either use the default pools, or start a pooled connection as necessary.
            * If you are using the default pools, be sure to use the (uncommented) pool settings from ```app.config```
            * ***If you use the default pools, then you will have to start up elasticsearch before the application, otherwise Bad Things™ will happen***, 
1. Start a pool, or a client process
	* ```erlasticsearch:start_client(<<"some_unique_name_here">>).```  
	       **or**
	* ```erlasticsearch:start_pool(<<"some_unique_name_here">>).```
1. Profit


***Note***   If you use a _client process_, then _all_ requests are serialized through this process. You might want it this way. Or you might not. Whatever.




WARNING
============
__**THE TESTS WILL CREATE AND DELETE INDICES IN WHATEVER ELASTICSEARCH INSTANCE YOU POINT THE CLIENT AT**__

__**THE TESTS WILL CREATE AND DELETE INDICES IN WHATEVER ELASTICSEARCH INSTANCE YOU POINT THE CLIENT AT**__

__**THE TESTS WILL CREATE AND DELETE INDICES IN WHATEVER ELASTICSEARCH INSTANCE YOU POINT THE CLIENT AT**__

__**!!!!!!!SERIOUSLY!!!!!!**__


__**YOU HAVE BEEN WARNED**__


TL;DR
============

1. Make sure you have ElasticSearch running.
2. Start up a connection
	* _Pool_ : ```erlasticsearch:start_pool(<<"some_unique_name_here">>).```  
	       **or**
	* _Client_ : ```erlasticsearch:start_client(<<"some_unique_name_here">>).``` 
       * In this case _All_ requests are serialized through this process. You might want it this way. Or you might not. Whatever.
1. Any JSON expected by ElasticSearch will need to go in as JSON  
   * For example --> ```<<"{\"settings\":{\"number_of_shards\":3}}">>```
1. Output returned by everything is in the form of ```[tuple()] | {error, Reason}```, i.e., either it is a list of tuples, or an error.  The tuple list will contain the following
   * **{status, Status}** <-- This is the REST code (200, 201, 404, etc.) 
   * **{body, Body}** <-- The body of the response from ElasticSearch. More on this next
   * **{result, Result}** <-- A boolean representing the result for the various boolean methods (```is_index```, ```is_doc```, etc.)
   * The Body of the response _from_ ElasticSearch - when it exists - will be JSON.  That said, ```binary_response``` in your ```connection_options``` is going to determine the form of the response.  
      * The default is ```binary_response = true```. In this case, you ```{body, Body}``` is just going to contain the entire payload from Elasticsearch as a single binary.
         * e.g. --> ```{body , <<"{\"ok\":true,\"acknowledged\":true}">>}```
      * If you set ```binary_response = false```, ```{body, Body}``` will contain the JSON as a decoded tuple-list (basically, what you get by running ```jsx:decode(Body)```)
         * ```{body , [ {<<"ok">> , true} , {<<"acknowledged">> , true} ] }```



Details
============

1. The thrift client is in the form of a _simple_one_for_one_ supervised client process (a _gen_server_).
1. You can explicitly start a (new) connection pool, and access ElasticSearch that way, as follows (e.g. if you need distinct pools for distinct DBs)
   * Start up a pool ---> ```erlasticsearch:start_pool(<<"some_unique_name_here">>).```
   * From that point, use ```{pool, <<"some_unique_name_here">>}``` as ```ServerRef``` 
1. You can explicitly start a (supervised!) client process, and access ElasticSearch that way, as follows (This is useful if you need to synchronize your requests, or some such)
   * Start up a client ---> ```erlasticsearch:start_client(<<"some_unique_name_here">>).```
   * Start a client process via ```start_client```.  From that point, any of the following three can serve as ```ServerRef``` in the accessors below
      * The client_name _ClientName_ that you passed in to ```start_client```
      * The pid _Pid_ that was returned from ```start_client```
      * The atom that the client process is registered under (accessible via ```erlasticsearch:registered_client_name/1```) 
1. Any JSON expected by ElasticSearch will need to go in as JSON  
   * For example --> ```<<"{\"settings\":{\"number_of_shards\":3}}">>```
1. Output returned by everything is in the form of ```[tuple()] | {error, Reason}```, i.e., either it is a list of tuples, or an error.  The tuple list will contain the following
   * **{status, Status}** <-- This is the REST code (200, 201, 404, etc.) 
   * **{body, Body}** <-- The body of the response from ElasticSearch. More on this next
   * **{result, Result}** <-- A boolean representing the result for the various boolean methods (```is_index```, ```is_doc```, etc.)
   * The Body of the response _from_ ElasticSearch - when it exists - will be JSON.  That said, ```binary_response``` in your ```connection_options``` is going to determine the form of the response.  
      * The default is ```binary_response = true```. In this case, you ```{body, Body}``` is just going to contain the entire payload from Elasticsearch as a single binary.
         * e.g. --> ```{body , <<"{\"ok\":true,\"acknowledged\":true}">>}```
      * If you set ```binary_response = false```, ```{body, Body}``` will contain the JSON as a decoded tuple-list (basically, what you get by running ```jsx:decode(Body)```)
         * ```{body , [ {<<"ok">> , true} , {<<"acknowledged">> , true} ] }```


***NOTE*** : Just remember, pools are always referenced as ```{pool, <<"some_name">>}``` and clients as ```<<"some_name">>```



Details
============
*[Supervisor tree diagram][sup_diagram]:*

[sup_diagram]: https://github.com/dieswaytoofast/erlasticsearch/blob/master/supervision_tree.jpg


Pool/Client Management
-----
These methods are available to start and stop the thrift pools/clients 
Note that once the application has been started, you can either

1. Start a client process via ```start_client```.  From that point, any of the following three can serve as ```ServerRef``` in the accessors below

   * The client_name _ClientName_ that you passed in to ```start_client```
   * The pid _Pid_ that was returned from ```start_client```
   * The atom that the client process is registered under (accessible via ```erlasticsearch:registered_client_name/1```)
1. Use the poolby version of the client, in which case you
   * Use ```start_pool``` and ```stop_pool```
   * Use ```{pool, <<"unique_pool_name">>}``` as ```ServerRef``` in teh accessors below   


Function | Parameters | Description
----- | ----------- | --------
start_client/1 | ClientName  | Start a client process reference-able as _ClientName_
start_client/2 | ClientName, Parameters | Start a client process reference-able as _ClientName_, with additional thrift parameters (also settable as _connection_options_ in ```app.config```)
stop_client/1 | ClientName  | Stop the client process references as _ClientName_
registered_client_name/1 | ClientName  | The atom that the thrift client process is registered under
start_pool/1 | PoolName  | Start a connection pool referenceable as _{pool, PoolName}_, with default ```pool_options``` and ```connection_options```
start_pool/2 | PoolName, PoolParameters | Start a connection pool referenceable as _{pool, PoolName}_, with custom ```pool_options``` and default ```connection_options```
start_pool/3 | PoolName, PoolParameters, ConnectionParameters | Start a connection pool referenceable as _{pool, PoolName}_, with custom ```pool_options``` and ```connection_options```
stop_pool/1 | PoolName  | Stop the connection pool referenced as _PoolName_

**EXAMPLES**

Using the _client_ based accessors  (note that ```bar2``` has ```{binary_response, false}```)


```erlang
erlasticsearch@pecorino)1> erlasticsearch:start_client(<<"bar1">>).
{ok,<0.178.0>}
erlasticsearch@pecorino)2> ok, Pid} = erlasticsearch:start_client(<<"bar2">>, [{thrift_options, [{framed, false}]}, {binary_response, false}]).
{ok,<0.182.0>}
erlasticsearch@pecorino)3> erlasticsearch:registered_client_name(<<"bar2">>).
'erlasticsearch_bar2.client'
erlasticsearch@pecorino)4> erlasticsearch:flush(<<"bar1">>).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"_shards\":{\"total\":0,\"successful\":0,\"failed\":0}}">>}]
erlasticsearch@pecorino)5> erlasticsearch:flush('erlasticsearch_bar2.client').
{ok,{restResponse,200,undefined,<<"{\"ok\":true,\"_shards\":{\"total\":552,\"successful\":276,\"failed\":0}}">>}}
erlasticsearch@pecorino)6> erlasticsearch:flush(Pid).
[{status,200},
 {body,[{<<"ok">>,true},
        {<<"_shards">>,
         [{<<"total">>,0},{<<"successful">>,0},{<<"failed">>,0}]}]}]
erlasticsearch@pecorino)7> erlasticsearch:flush(list_to_pid("<0.182.0>")).
[{status,200},
 {body,[{<<"ok">>,true},
        {<<"_shards">>,
         [{<<"total">>,0},{<<"successful">>,0},{<<"failed">>,0}]}]}]
erlasticsearch@pecorino)8> erlasticsearch:stop_client(<<"bar1">>).
ok
erlasticsearch@pecorino)9> erlasticsearch:stop_client(<<"bar2">>).
ok
```

Using _poolboy_ based accessors  (note that ```bar2``` has ```{binary_response, false}```)

```erlang
erlasticsearch@pecorino)1> erlasticsearch:start_pool(<<"bar0">>).
{ok,<0.83.0>}
erlasticsearch@pecorino)2> erlasticsearch:start_pool(<<"bar1">>, [{size, 10}, {max_overflow, 20}]).
{ok,<0.111.0>}
erlasticsearch@pecorino)3> erlasticsearch:start_pool(<<"bar2">>, [{size, 10}, {max_overflow, 20}], [{thrift_host, "localhost"}, {binary_response, false}]).  
{ok,<0.135.0>}
erlasticsearch@pecorino)4> erlasticsearch:flush({pool, <<"bar1">>}).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"_shards\":{\"total\":0,\"successful\":0,\"failed\":0}}">>}]
erlasticsearch@pecorino)5> erlasticsearch:flush({pool, <<"bar3">>}).
[{status,200},
 {body,[{<<"ok">>,true},
        {<<"_shards">>,
         [{<<"total">>,0},{<<"successful">>,0},{<<"failed">>,0}]}]}]
```


Index CRUD
-----
These methods are available to perform CRUD activities on Indexes (kinda, sorta, vaguely the equivalent of Databases)

Function | Parameters | Description
----- | ----------- | --------
create_index/2 | ServerRef, IndexName  | Creates the Index called _IndexName_
create_index/3 | ServerRef, IndexName, Parameters | Creates the Index called _IndexName_, with additional options as specified [here](http://www.elasticsearch.org/guide/reference/api/admin-indices-create-index/)
delete_index/2 | ServerRef, IndexName  | Deletes the Index called _IndexName_
is_index/2 | ServerRef, IndexName  | Checks if the Index called _IndexName_ exists. (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```)
is_type/3 | ServerRef, IndexName, TypeName  | Checks if the Type called _TypeName exists in the index _IndexName_. (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```), as well as a list of types (e.g. ```[<<"type1">>, <<"type2">>]```)
open_index/2 | ServerRef, IndexName  | Opens the Index called _IndexName_
close_index/2 | ServerRef, IndexName  | Closes the Index called _IndexName_



**EXAMPLES**

(note that ```bar2``` has ```{binary_response, false}```)

```erlang
erlasticsearch@pecorino)1> erlasticsearch:start_client(<<"bar">>).
{ok,<0.178.0>}
erlasticsearch@pecorino)2> erlasticsearch:start_client(<<"bar2">>, [{binary_response, false}]).
{ok,<0.182.0>}
erlasticsearch@pecorino)3> erlasticsearch:create_index(<<"bar">>, <<"foo2">>).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"acknowledged\":true}">>}]
erlasticsearch@pecorino)4> erlasticsearch:create_index(<<"bar2">>, <<"foo3">>, <<"{\"settings\":{\"number_of_shards\":3}}">>).
[{status,200},
 {body,[{<<"ok">>,true},{<<"acknowledged">>,true}]}]
erlasticsearch@pecorino)5> erlasticsearch:create_index(<<"bar">>, <<"foo4">>, <<"{\"settings\":{\"number_of_shards\":3}}">>).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"acknowledged\":true}">>}]
```
```erlang
erlasticsearch@pecorino)6> erlasticsearch:delete_index(<<"bar">>, <<"foo2">>).                                               
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"acknowledged\":true}">>}]
erlasticsearch@pecorino)7> erlasticsearch:stop_client(<<"bar">>).
ok
```
```erlang
erlasticsearch@pecorino)8> erlasticsearch:is_index({pool, <<"an_erlasticsearch_pool">>}, <<"foo3">>).    
[{result,<<"false">>},{status,<<"404">>}]
erlasticsearch@pecorino)9> erlasticsearch:is_index({pool, <<"an_erlasticsearch_pool">>}, <<"foo4">>).
[{result,<<"true">>},{status,<<"200">>}]
erlasticsearch@pecorino)10> erlasticsearch:is_index({pool, <<"an_erlasticsearch_pool">>}, [<<"foo3">>, <<"foo4">>]).
[{result,<<"true">>},{status,<<"200">>}]
erlasticsearch@pecorino)11> erlasticsearch:is_index({pool, <<"an_erlasticsearch_pool">>}, <<"no_such_index">>).
[{result,<<"false">>},{status,<<"404">>}]
erlasticsearch@pecorino)12> erlasticsearch:is_type({pool, <<"an_erlasticsearch_pool">>}, <<"foo3">>, <<"existing_type_1">>).
[{result,<<"true">>},{status,<<"200">>}]
erlasticsearch@pecorino)13> erlasticsearch:is_type({pool, <<"an_erlasticsearch_pool">>}, [<<"foo3">>, <<"foo4">>], <<"existing_type_1">>).
[{result,<<"true">>},{status,<<"200">>}]
erlasticsearch@pecorino)14> erlasticsearch:is_type({pool, <<"an_erlasticsearch_pool">>}, [<<"foo3">>, <<"foo4">>], [<<"existing_type_1">>, <<"existing_type_2">>]).
[{result,<<"true">>},{status,<<"200">>}]
```
```erlang
erlasticsearch@pecorino)15> erlasticsearch:open_index({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"acknowledged\":true}">>}]
erlasticsearch@pecorino)16> erlasticsearch:close_index({pool, <<"an_erlasticsearch_pool">>}, <<"foo4">>).                  
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"acknowledged\":true}">>}]
```



Document CRUD
-----
These methods are available to perform CRUD activities on actual documents 

(**Note**: _ServerRef_ is either a _Client Name/Reference_, or _{pool, PoolName}_)

Function | Parameters | Description
----- | ----------- | --------
insert_doc/5 | ServerRef, IndexName, Type, Id, Doc  | Creates the Doc under _IndexName_, with type _Type_, and id _Id_
insert_doc/6 | ServerRef, IndexName, Type, Id, Doc, Params  | Creates the Doc under _IndexName_, with type _Type_, and id _Id_, and passes the tuple-list _Params_ to ElasticSearch
is_doc/4 | ServerRef, IndexName, Type, Id  | Checks if the Doc under _IndexName_, with type _Type_, and id _Id_ exists
get_doc/4 | ServerRef, IndexName, Type, Id  | Gets the Doc under _IndexName_, with type _Type_, and id _Id_
get_doc/5 | ServerRef, IndexName, Type, Id, Params  | Gets the Doc under _IndexName_, with type _Type_, and id _Id_, and passes the tuple-list _Params_ to ElasticSearch
mget_doc/2 | ServerRef, Doc  | Gets documents from the ElasticSearch cluster based on the Index(s), Type(s), and Id(s) in _Doc_
mget_doc/3 | ServerRef, IndexName, Doc  | Gets documents from the ElasticSearch cluster index _IndexName_ based on the Type(s), and Id(s) in _Doc_
mget_doc/4 | ServerRef, IndexName, TypeName, Doc  | Gets documents from the ElasticSearch cluster index _IndexName_, with type _TypeName_, based on the Id(s) in _Doc_
delete_doc/4 | ServerRef, IndexName, Type, Id  | Deleset the Doc under _IndexName_, with type _Type_, and id _Id_
delete_doc/5 | ServerRef, IndexName, Type, Id, Params  | Deletes the Doc under _IndexName_, with type _Type_, and id _Id_, and passes the tuple-list _Params_ to ElasticSearch
count/2 | ServerRef, Doc | Counts the docs in the cluster based on the search in _Doc_. (note that if _Doc_ is empty, you get a count of all the docs in the cluster)
count/3 | ServerRef, Doc, Params | Counts the docs in the cluster based on the search in _Doc_, using _Params_.  Note that either _Doc_ or _Params_ can be empty, but clearly not both :-)
count/4 | ServerRef, IndexName, Doc, Params | Counts the docs in the cluster based on the search in _Doc_, associated with the index _IndexName_, using _Params_  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```. This list can also be empty - ```[]```)
count/5 | ServerRef, IndexName, TypeName, Doc, Params | Counts the docs in the cluster based on the search in _Doc_, associated with the index _IndexName_, and type _TypeName_ using _Params_  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```, as well as a list of types (e.g. ) ```[<<"type1">>, <<"type2">>]```. Each of these lists can also be empty - ```[]```)
delete_by_query/2 | ServerRef, Doc | Deletes the docs in the cluster based on the search in _Doc_. (note that if _Doc_ is empty, you get a count of all the docs in the cluster)
delete_by_query/3 | ServerRef, Doc, Params | Deletes the docs in the cluster based on the search in _Doc_, using _Params_.  Note that either _Doc_ or _Params_ can be empty, but clearly not both :-)
delete_by_query/4 | ServerRef, IndexName, Doc, Params | Deletes the docs in the cluster based on the search in _Doc_, associated with the index _IndexName_, using _Params_  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```. This list can also be empty - ```[]```)
delete_by_query/5 | ServerRef, IndexName, TypeName, Doc, Params | Deletes the docs in the cluster based on the search in _Doc_, associated with the index _IndexName_, and type _TypeName_ using _Params_  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```, as well as a list of types (e.g. ) ```[<<"type1">>, <<"type2">>]```. Each of these lists can also be empty - ```[]```)




_Note_: 

1. For both ```insert_doc/4``` and ```insert_doc/5```, sending in ```undefined``` as the ```Id``` will result in ElasticSearch generating an Id for the document.  This Id will be returned as part of the result...
2. Yes, the order of the arguments to mget_doc/[2,3,4] is weird.  Its just that ElasticSearch is slightly strange in this one...

**EXAMPLES**

```erlang
erlasticsearch@pecorino)4> erlasticsearch:start_client(<<"bar">>).
{ok,<0.178.0>}
erlasticsearch@pecorino)5> erlasticsearch:start_client(<<"bar2">>, [{binary_response, false}]).
{ok,<0.182.0>}
erlasticsearch@pecorino)6> erlasticsearch:insert_doc(<<"bar">>, <<"index1">>, <<"type1">>, <<"id1">>, <<"{\"some_key\":\"some_val\"}">>).
[{status,<<"201">>},
 {body,<<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1}">>}]
erlasticsearch@pecorino)7> erlasticsearch:insert_doc(<<"bar">>, <<"index2">>, <<"type3">>, <<"id2">>, <<"{\"some_key\":\"some_val\"}">>, [{'_ttl', '1d'}]). 
[{status,<<"201">>},
 {body,<<"{\"ok\":true,\"_index\":\"index2\",\"_type\":\"type3\",\"_id\":\"id2\",\"_version\":1}">>}]
erlasticsearch@pecorino)8> erlasticsearch:insert_doc(<<"bar2">>, <<"index3">>, <<"type3">>, undefined, <<"{\"some_key\":\"some_val\"}">>).
[{status,201},
 {body,[{<<"ok">>,true},
        {<<"_index">>,<<"index3">>},
        {<<"_type">>,<<"type3">>},
        {<<"_id">>,<<"z9M78se6SuKsZ0lYlybAwg">>},
        {<<"_version">>,1}]}]
```
```erlang
erlasticsearch@pecorino)9> erlasticsearch:is_doc(<<"bar">>, <<"index1">>, <<"type1">>, <<"id1">>).
[{result,<<"true">>},{status,<<"200">>}]
```
```erlang
erlasticsearch@pecorino)9> erlasticsearch:get_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"id1">>).
[{status,<<"200">>},
 {body,<<"{\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1,\"exists\":true, \"_source\" : {\"some_key\":\""...>>}]
erlasticsearch@pecorino)10> erlasticsearch:get_doc(<<"bar2">>, <<"index1">>, <<"type1">>, <<"id1">>, [{fields, foobar}]).                             [{status,200},
 {body,[{<<"_index">>,<<"index1">>},
        {<<"_type">>,<<"type1">>},
        {<<"_id">>,<<"id1">>},
        {<<"_version">>,1},
        {<<"exists">>,true}]}]
erlasticsearch@pecorino)11> erlasticsearch:get_doc(<<"bar2">>, <<"index1">>, <<"type1">>, <<"id1">>, [{fields, some_key}]).
[{status,200},
 {body,[{<<"_index">>,<<"index1">>},
        {<<"_type">>,<<"type1">>},
        {<<"_id">>,<<"id1">>},
        {<<"_version">>,1},
        {<<"exists">>,true},
        {<<"fields">>,[{<<"some_key">>,<<"some_val">>}]}]}]
```
```erlang
erlasticsearch@pecorino)12> erlasticsearch:delete_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"id1">>).                   
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"found\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":2}">>}]
```
```erlang
erlasticsearch@pecorino)13> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"201">>},
 {body,<<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"1\",\"_version\":1}">>}]
erlasticsearch@pecorino)14> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index2">>, <<"type2">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"201">>},
 {body,<<"{\"ok\":true,\"_index\":\"index2\",\"_type\":\"type2\",\"_id\":\"1\",\"_version\":1}">>}]
erlasticsearch@pecorino)15> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, <<>>).
[{status,<<"200">>},
 {body,<<"{\"count\":5,\"_shards\":{\"total\":15,\"successful\":15,\"failed\":0}}">>}]
erlasticsearch@pecorino)16> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, <<>>, [{q, <<"key_1:value_1">>}]).
[{status,<<"200">>},
 {body,<<"{\"count\":2,\"_shards\":{\"total\":15,\"successful\":15,\"failed\":0}}">>}]
erlasticsearch@pecorino)17> erlasticsearch:count(<<"bar2">>, <<"{\"term\":{\"key_1\":\"value_1\"}}">>, []).                               
[{status,200},
 {body,[{<<"count">>,2},
        {<<"_shards">>,
         [{<<"total">>,15},
          {<<"successful">>,15},
          {<<"failed">>,0}]}]}]
erlasticsearch@pecorino)18> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>],<<>>, [{q, <<"key_1:value_1">>}]).
[{status,<<"200">>},
 {body,<<"{\"count\":1,\"_shards\":{\"total\":5,\"successful\":5,\"failed\":0}}">>}]
erlasticsearch@pecorino)19> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>,<<"index2">>],[<<"type1">>, <<"type2">>],<<>>, [{q, <<"key_1:value_1">>}]).
erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>,<<"index2">>],[<<"type1">>, <<"type2">>],<<>>, [{q, <<"key_1:value_1">>}]).
[{status,<<"200">>},
 {body,<<"{\"count\":2,\"_shards\":{\"total\":10,\"successful\":10,\"failed\":0}}">>}]
```              
```erlang
erlasticsearch@pecorino)20> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"1\",\"_version\":2}">>}]
erlasticsearch@pecorino)21> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index2">>, <<"type2">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"200">>},
 {body,<<"{\"ok\":true,\"_index\":\"index2\",\"_type\":\"type2\",\"_id\":\"1\",\"_version\":2}">>}]
erlasticsearch@pecorino)22> erlasticsearch:delete_by_query({pool, <<"an_erlasticsearch_pool">>}, <<"{\"term\":{\"key_1\":\"value_1\"}}">>).                  [{status,<<"201">>},
 {body,<<"{\"ok\":true,\"_indices\":{\"index_137402104\":{\"_shards\":{\"total\":5,\"successful\":5,\"failed\":0}},\""...>>}]
erlasticsearch@pecorino)23> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, <<"{\"term\":{\"key_1\":\"value_1\"}}">>).
[{status,<<"201">>},
 {body, <<"{\"count\":0,\"_shards\":{\"total\":245,\"successful\":245,\"failed\":0}}">>}]
```
```erlang
erlasticsearch@pecorino)24> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"201">>},
 {body, <<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"1\",\"_version\":1}">>}]
erlasticsearch@pecorino)25> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index2">>, <<"type2">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_index\":\"index2\",\"_type\":\"type2\",\"_id\":\"1\",\"_version\":1}">>}]
erlasticsearch@pecorino)26> erlasticsearch:delete_by_query({pool, <<"an_erlasticsearch_pool">>}, <<>>, [{q, <<"key1:value1">>}]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_indices\":{\"index_137402104\":{\"_shards\":{\"total\":5,\"successful\":5,\"failed\":0}},\""...>>}]
erlasticsearch@pecorino)27> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, <<"{\"term\":{\"key_1\":\"value_1\"}}">>).
[{status,<<"200">>},
 {body, <<"{\"count\":0,\"_shards\":{\"total\":245,\"successful\":245,\"failed\":0}}">>}]
```
```erlang
erlasticsearch@pecorino)28> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"201">>},
 {body, <<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"1\",\"_version\":1}">>}]
erlasticsearch@pecorino)29> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index2">>, <<"type2">>, <<"1">>, <<"{\"key_1\":\"value_1\"},{\"key_2\":\"value_2\"}">>).
[{status,<<"201">>},
 {body, <<"{\"ok\":true,\"_index\":\"index2\",\"_type\":\"type2\",\"_id\":\"1\",\"_version\":1}">>}]
erlasticsearch@pecorino)30> erlasticsearch:delete_by_query({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>, <<"index2">>], [<<"type1">>, <<"type2">>], <<>>, [{q, <<"key1:value1">>}]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_indices\":{\"index1\":{\"_shards\":{\"total\":5,\"successful\":5,\"failed\":0}},\"index2\":{"...>>}]
erlasticsearch@pecorino)31> erlasticsearch:count({pool, <<"an_erlasticsearch_pool">>}, <<"{\"term\":{\"key_1\":\"value_1\"}}">>).
[{status,<<"200">>},
 {body, <<"{\"count\":0,\"_shards\":{\"total\":245,\"successful\":245,\"failed\":0}}">>}]
```
```erlang
erlasticsearch@pecorino)32> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"id1">>, <<"{\"key_1\":\"value_1\"}">>).
[{status,<<"201">>},
 {body, <<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1}">>}]
erlasticsearch@pecorino)33>  erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index2">>, <<"type2">>, <<"id2">>, <<"{\"key_2\":\"value_2\"}">>).
[{status,<<"201">>},
 {body, <<"{\"ok\":true,\"_index\":\"index2\",\"_type\":\"type2\",\"_id\":\"id2\",\"_version\":1}">>}]
erlasticsearch@pecorino)34> FullDoc = jsx:encode([{docs, [[{<<"_index">>, <<"index1">>},{<<"_type">>, <<"type1">>},{<<"_id">>, <<"id1">>}], [{<<"_index">>, <<"index2">>},{<<"_type">>, <<"type2">>},{<<"_id">>, <<"id2">>}]]}]).
<<"{\"docs\":[{\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\"},{\"_index\":\"index2\",\"_type\":\"type2\",\"_id\":\"id2\"}]}">>
erlasticsearch@pecorino)35> erlasticsearch:mget_doc({pool, <<"an_erlasticsearch_pool">>}, FullDoc).                                                         [{status,<<"200">>},
 {body, <<"{\"docs\":[{\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1,\"exists\":true, \"_source"...>>}]
erlasticsearch@pecorino)36> TypeDoc = jsx:encode([{docs, [[{<<"_type">>, <<"type1">>},{<<"_id">>, <<"id1">>}], [{<<"_type">>, <<"type2">>},{<<"_id">>, <<"id2">>}]]}]).
<<"{\"docs\":[{\"_type\":\"type1\",\"_id\":\"id1\"},{\"_type\":\"type2\",\"_id\":\"id2\"}]}">>
erlasticsearch@pecorino)37> erlasticsearch:mget_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, TypeDoc).
[{status,<<"200">>},
 {body, <<"{\"docs\":[{\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1,\"exists\":true, \"_source"...>>}]
erlasticsearch@pecorino)38> IdDoc = jsx:encode([{docs, [[{<<"_id">>, <<"id1">>}], [{<<"_id">>, <<"id2">>}]]}]).
<<"{\"docs\":[{\"_id\":\"id1\"},{\"_id\":\"id2\"}]}">>
erlasticsearch@pecorino)39> erlasticsearch:mget_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, IdDoc).
[{status,<<"201">>},
 {body, <<"{\"docs\":[{\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1,\"exists\":true, \"_source"...>>}]           
```



Search
-----
API to perform searches against ElasticSearch (this _is_ why you are using ElasticSearch, right?)

(**Note**: _ServerRef_ is either a _Client Name/Reference_, or _{pool, PoolName}_)

Function | Parameters | Description
----- | ----------- | --------
search/4 | ServerRef, IndexName, Type, Doc  | Searches the index _IndexName_, with type _Type_ for the JSON query embedded in _Doc_
search/5 | ServerRef, IndexName, Type, Doc, Params  | Searches the index _IndexName_, with type _Type_ for the JSON query embedded in _Doc_, and passes the tuple-list _Params_ to ElasticSearch



**EXAMPLES**

```erlang
erlasticsearch@pecorino)2> erlasticsearch:insert_doc({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<"id1">>, <<"{\"some_key\":\"some_val\"}">>).
[{status,<<"201">>},
 {body, <<"{\"ok\":true,\"_index\":\"index1\",\"_type\":\"type1\",\"_id\":\"id1\",\"_version\":1}">>}]
erlasticsearch@pecorino)3> erlasticsearch:search({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>, <<"type1">>, <<>>, [{q, "some_key:some_val"}]).     
[{status,<<"200">>},
 {body, <<"{\"took\":1,\"timed_out\":false,\"_shards\":{\"total\":5,\"successful\":5,\"failed\":0},\"hits\":{\"total\":"...>>}]
```

Index Helpers
-----
A bunch of functions that do "things" to indices (flush, refresh, etc.)

(**Note**: _ServerRef_ is either a _Client Name/Reference_, or _{pool, PoolName}_)

Function | Parameters | Description
----- | ----------- | --------
flush/1 | ServerRef  | Flushes all the indices
flush/2 | ServerRef, Index | Flushes the index _IndexName_.  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]```)
optimize/1 | ServerRef  | Optimizes all the indices
optimize/2 | ServerRef, Index | Optimizes the index _IndexName_.  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]``` This list can also be empty - ```[]```)
segments/1 | ServerRef  | Provides segment information for all the indices in the cluster
segments/2 | ServerRef, Index | Provides segment information for the index _IndexName_.  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]``` This list can also be empty - ```[]```)
refresh/1 | ServerRef  | Refreshes all the indices
refresh/2 | ServerRef, Index | Refreshes the index _IndexName_.  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]``` This list can also be empty - ```[]```)
status/2 | ServerRef, Index | Returns the status of index _IndexName_.  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]``` This list can also be empty - ```[]```)
clear_cache/1 | ServerRef  | Clears all the caches in the cluster
clear_cache/2 | ServerRef, Index | Clears all the caches associated with the index _IndexName_.  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]``` This list can also be empty - ```[]```)
clear_cache/3 | ServerRef, Index, params | Clears all the caches associated with the index _IndexName_, using _Params_  (Note that a list of Indices can also be sent in (e.g., ```[<<"foo">>, <<"bar">>]``` This list can also be empty - ```[]```)


**EXAMPLES**

```erlang
erlasticsearch@pecorino)2> erlasticsearch:refresh({pool, <<"an_erlasticsearch_pool">>}).                                                                                   [{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":552,\"successful\":276,\"failed\":0}}">>}] 
erlasticsearch@pecorino)3> erlasticsearch:refresh({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>).                                                                        [{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":10,\"successful\":5,\"failed\":0}}">>}]
erlasticsearch@pecorino)4> erlasticsearch:refresh({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>, <<"index2">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":16,\"successful\":8,\"failed\":0}}">>}]
```
```erlang
erlasticsearch@pecorino)5> erlasticsearch:flush({pool, <<"an_erlasticsearch_pool">>}).                                                                                   [{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":552,\"successful\":276,\"failed\":0}}">>}] 
erlasticsearch@pecorino)6> erlasticsearch:refresh({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>).                                                                        [{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":10,\"successful\":5,\"failed\":0}}">>}]
erlasticsearch@pecorino)7> erlasticsearch:refresh({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>, <<"index2">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":16,\"successful\":8,\"failed\":0}}">>}] 
```
```erlang
erlasticsearch@pecorino)8> erlasticsearch:status({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":6,\"successful\":3,\"failed\":0},\"indices\":{\"index1\":{\"index\":{\"prim"...>>}]
erlasticsearch@pecorino)9> erlasticsearch:status({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>, <<"index2">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":16,\"successful\":8,\"failed\":0},\"indices\":{\"index2\":{\"index\":{\"pri"...>>}]
erlasticsearch@pecorino)10> erlasticsearch:optimize({pool, <<"an_erlasticsearch_pool">>}, <<"index1">>).     
[{status,<<"200">>},
 {body, <"{\"ok\":true,\"_shards\":{\"total\":10,\"successful\":5,\"failed\":0}}">>}]
erlasticsearch@pecorino)11> erlasticsearch:optimize({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>, <<"index2">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":16,\"successful\":8,\"failed\":0}}">>}]
erlasticsearch@pecorino)12> erlasticsearch:optimize({pool, <<"an_erlasticsearch_pool">>}).                         
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":692,\"successful\":346,\"failed\":0}}">>}]                 
```
```erlang
erlasticsearch@pecorino)13> erlasticsearch:segments({pool, <<"an_erlasticsearch_pool">>}).                                     
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":170,\"successful\":85,\"failed\":0},\"indices\":{\"test_1\":{\"shards\":"...>>}]
erlasticsearch@pecorino)14> erlasticsearch:segments({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":10,\"successful\":5,\"failed\":0},\"indices\":{\"index1\":{\"shards\":{\""...>>}]
erlasticsearch@pecorino)15> erlasticsearch:segments({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>, <<"index2">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":20,\"successful\":10,\"failed\":0},\"indices\":{\"index1\":{\"shards\":{"...>>}]
```
```erlang
erlasticsearch@pecorino)16> erlasticsearch:clear_cache({pool, <<"an_erlasticsearch_pool">>}).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":130,\"successful\":65,\"failed\":0}}">>}]
erlasticsearch@pecorino)17> erlasticsearch:clear_cache({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":10,\"successful\":5,\"failed\":0}}">>}]
erlasticsearch@pecorino)18> erlasticsearch:clear_cache({pool, <<"an_erlasticsearch_pool">>}, [<<"index1">>], [{filter, true}]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":10,\"successful\":5,\"failed\":0}}">>}]
erlasticsearch@pecorino)19> erlasticsearch:clear_cache({pool, <<"an_erlasticsearch_pool">>}, [], [{filter, true}]).            
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"_shards\":{\"total\":140,\"successful\":70,\"failed\":0}}">>}]        
```


Cluster Helpers
-----
A bunch of functions that do "things" to clusters (health, etc.)

(**Note**: _ServerRef_ is either a _Client Name/Reference_, or _{pool, PoolName}_)

Function | Parameters | Description
----- | ----------- | --------
health/1 | ServerRef  | Reports the health of the cluster
state/1 | ServerRef  | Reports the state of the cluster
state/2 | ServerRef, Params  | Reports the state of the cluster, with optional parameters
nodes_info/1 | ServerRef  | Reports the state of all the nodes in the cluster
nodes_info/2 | ServerRef, NodeName  | Reports the state of the node _NodeName_ in the cluster. (Note that a list of Nodes can also be sent in (e.g., ```[<<"node1">>, <<"node2">>]``` This list can also be empty - ```[]```)
nodes_info/3 | ServerRef, NodeName, Params  | Reports the state of the node _NodeName_ in the cluster, with optional _Params_. (Note that a list of Nodes can also be sent in (e.g., ```[<<"node1">>, <<"node2">>]``` This list can also be empty - ```[]```)
nodes_stats/1 | ServerRef  | Reports stats on all the nodes in the cluster
nodes_stats/2 | ServerRef, NodeName  | Reports the stats of the node _NodeName_ in the cluster. (Note that a list of Nodes can also be sent in (e.g., ```[<<"node1">>, <<"node2">>]```)
nodes_stats/3 | ServerRef, NodeName, Params  | Reports the stats of the node _NodeName_ in the cluster, with optional _Params_. (Note that a list of Nodes can also be sent in (e.g., ```[<<"node1">>, <<"node2">>]``` This list can also be empty - ```[]```)


**EXAMPLES**
```erlang
erlasticsearch@pecorino)1> erlasticsearch:start_client(<<"bar">>).
{ok,<0.178.0>}
erlasticsearch@pecorino)2> erlasticsearch:refresh(<<"bar">>).                                                                                   {ok,{restResponse,200,undefined,                                                                                                                                               <<"{\"ok\":true,\"_shards\":{\"total\":552,\"successful\":276,\"failed\":0}}">>}]
erlasticsearch@pecorino)3> erlasticsearch:health(<<"bar">>).                          
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"status\":\"yellow\",\"timed_out\":false,\"number_of_nodes\""...>>}]
erlasticsearch@pecorino)1> erlasticsearch:stop_client(<<"bar">>).
ok
```
```erlang
erlasticsearch@pecorino)4> erlasticsearch:state({pool, <<"an_erlasticsearch_pool">>}).
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"master_node\":\"7k3ViuT5SQ67ayWsF1y8hQ\",\"blocks\":{\"ind"...>>}]
erlasticsearch@pecorino)5> erlasticsearch:state({pool, <<"an_erlasticsearch_pool">>}, [{filter_nodes, true}]).
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"blocks\":{\"indices\":{\"index1\":{\"4\":{\"description\":\"inde"...>>}]
```
```erlang
erlasticsearch@pecorino)6> erlasticsearch:nodes_info({pool, <<"an_erlasticsearch_pool">>}).                   
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"name\":\""...>>}]
erlasticsearch@pecorino)7> erlasticsearch:nodes_info({pool, <<"an_erlasticsearch_pool">>}, <<"node1">>).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"name\":\""...>>}]
erlasticsearch@pecorino)8> erlasticsearch:nodes_info({pool, <<"an_erlasticsearch_pool">>}, [<<"node1">>]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"name\":\""...>>}]
erlasticsearch@pecorino)9> erlasticsearch:nodes_info({pool, <<"an_erlasticsearch_pool">>}, [<<"node1">>], [{os, true}, {process, true}]).
[{status,<<"200">>},
 {body, <<"{\"ok\":true,\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"name\":\""...>>}]
```
```erlang
erlasticsearch@pecorino)10> erlasticsearch:nodes_stats({pool, <<"an_erlasticsearch_pool">>}).                              
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"timestamp\":136865"...>>}]
erlasticsearch@pecorino)11> erlasticsearch:nodes_stats({pool, <<"an_erlasticsearch_pool">>}, <<"node1">>).
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"timestamp\":136865"...>>}]
erlasticsearch@pecorino)12> erlasticsearch:nodes_stats({pool, <<"an_erlasticsearch_pool">>}, [<<"node1">>]).
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"timestamp\":136865"...>>}]
erlasticsearch@pecorino)13> erlasticsearch:nodes_stats({pool, <<"an_erlasticsearch_pool">>}, [<<"node1">>], [{process, true}, {transport, true}]).
[{status,<<"200">>},
 {body, <<"{\"cluster_name\":\"elasticsearch_mahesh\",\"nodes\":{\"node1\":{\"timestamp\":136865"...>>}]
```



Credits
=======

Thanks to [Paul Oliver](https://github.com/puzza007) for helping with the [poolboy](https://github.com/devinus/poolboy) implementation

This is _not_ to be confused with [erlastic_search](https://github.com/tsloughter/erlastic_search) by [Tristan Sloughter](https://github.com/tsloughter), which is HTTP/REST based, and almost certainly did not involve quite this level of head-thumping associated w/ figuring out how Thrift works…

(Yes, this is a _Credit_)
