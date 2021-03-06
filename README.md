# NodeDBI

NodeDBI is a LibDBI interface for Node.js.

For the source code please visit [the github repository](https://github.com/danieloneill/nodedbi).

In addition to providing a traditional interface for SQL database access, it also offers developers the ability of paging on results programmatically and storing result handles to a session as shown below.

It's somewhat complete, and of course contributions are much appreciated.

LibDBI and development headers are required.

On Debian or Ubuntu, **apt-get install libdbi-dev.**

Then run **node-gyp configure build** to build the module and **node-gyp install** to install it.

## Example

I've put together a couple Gists that may be useful:

* [node.js + nodedbi, convenience methods](https://gist.github.com/danieloneill/2605640f020c89fb806a)
* [Nodejs + libdbi + paging results](https://gist.github.com/danieloneill/d069be8e02e852008cbd)

```javascript
#!/usr/bin/nodejs

var mod = require('nodedbi');

var args = { 'host':'localhost', 'port':3306, 'username':'root', 'password':'', 'type':'mysql', 'dbname':'test' };

var obj = new mod.DBConnection( args );

var q = obj.query("SELECT * FROM users WHERE username=%1 OR id=%2", ['doneill', 6]);
if( !q )
{
        console.log("Query failed!");
        return;
}

// Simple:
var results = q.toArray();

// Doing the same thing manually:
var rc = q.count();
console.log("Row count: "+rc);

var fc = q.fieldCount();
console.log("Field count: "+q.fieldCount());

var fna = [];
for( var x=0; x < fc; x++ )
{
        var fn = q.fieldName(x+1);
        console.log("Field name("+(x+1)+"): "+fn);
        fna.push( fn );
}

for( var y=0; y < rc; y++ )
{
        q.seek(y+1);
        for( var x=0; x < fc; x++ )
        {
                var v = q.value(x+1);
                console.log( y+" ["+fna[x]+"] = ["+v+"]" );
        }
}

```

# Methods

## DBConnection( args )
`Create a new database connection and connect.`

Must be called with *new* qualifier, eg:

```javascript
var NodeDBI = require('nodedbi');
var db = new NodeDBI.DBConnection(args);
```

A **type** parameter is required. Other parameters are database specific. For database specific driver options see [the libdbi-drivers documentation](http://libdbi-drivers.sourceforge.net/docs.html).

* `args` - Object containing LibDBI-specific connection parameters in key:value sets.
* `Returns`: A database object, or **undefined** and an Exception on failure.

---

## DBConnection::query(queryString, [parameters])
`Create and execute a database query.`

* `queryString` - Query string with placeholders enumerated as %1, %2, etc.
* `parameters` - Optional array of query placeholder values.
* `Returns`: A DBQuery object, or **false** if the query fails for any reason.

---

## DBConnection::lastError()
`Retrieve the latest error as a string.`

* `Returns`: The latest database error as a string.

---

## DBConnection::lastErrorCode()
`Retrieve the latest error.`

* `Returns`: The latest database error enumerated as below. See [LibDBI's documentation for a comprehensive list](http://libdbi.sourceforge.net/docs/programmers-guide/connerrors.html). *(Enumerations are members of the imported nodedbi module. For example, if imported with* **var DBI = require('nodedbi');** *one would reference an error as* **if( db.lastErrorCode() == DBI.DBI_ERROR_CLIENT ) ... ** *or similar.*)
  * DBI_ERROR_USER
  * DBI_ERROR_DBD
  * ...
  * DBI_ERROR_NONE

---

## Connection::lastInsertId([name])
`Fetch the row ID generated by the last INSERT command. The row ID is most commonly generated by an auto-incrementing column in the table.`

* `name` - The name of the sequence, or NULL if the database engine does not use explicit sequences.
* `Returns`: A value corresponding to the ID that was created by the last INSERT command. If the database engine does not support sequences, the function returns 0.

---

## Query()
`This object is returned by` **DBConnection::query** `and cannot be instantiated on its own.`

---

## Query::count()
`Row count of result set.`

* `Returns`: A count of the returned rows in a SELECT or similar query.

---

## Query::fieldCount()
`Column count of the result set.`

* `Returns`: A count of the returned columns (fields) in a SELECT or similar query.

---

## Query::fieldName(column)
`Fetch the field name of a specific column index. Column indexes in LibDBI begin at` **1**`, not 0.`

* `column` - The column index in the result object. Column indexes in LibDBI begin at **1**, not 0.
* `Returns`: The name of the field at the specified index or **undefined** and an exception if the index is invalid.

---

## Query::fieldIndex(fieldName)
`Fetch the column index of a specific field name. Column indexes in LibDBI begin at` **1**`, not 0.`

* `fieldName` - The field name in the result object.
* `Returns`: The column index with the specified field name or **undefined** and an exception if the field name isn't found. Column indexes in LibDBI begin at **1**, not 0.

---

## Query::seek(row)
`Seek to a specified row in the resultset. Row indexes in LibDBI begin at` **1**`, not 0.`

* `row` - The row index in the result object. Row indexes in LibDBI begin at **1**, not 0.
* `Returns`: **true** on success, **false** or an exception on failure.

---

## Query::previousRow()
`Seek the cursor to the previous row in the resultset.`

* `Returns`: **true** on success, **false** if already at the first result in the set.

---

## Query::nextRow()
`Seek the cursor to the next row in the resultset.`

* `Returns`: **true** on success, **false** if already at the last result in the set.

---

## Query::currentRow()
`Fetch the current row index of the resultset cursor. Row indexes in LibDBI begin at` **1**`, not 0.`

* `Returns`: The current row index. Row indexes in LibDBI begin at **1**, not 0.

---

## Query::value(field)
`Fetch the value of the specified field on the current row index of the resultset cursor. Row and column indexes in LibDBI begin at` **1**`, not 0.`

**field** can be either a numeric column index or the field name.

Values are converted to the best Javascript type possible to avoid loss of precision. Binary data is returned as a [Buffer object](https://nodejs.org/api/buffer.html).

* `Returns`: The value.

---

## Query::begin()
`Starts a transaction.`

* `Returns`: **true** on success, **false** if the database cannot or will not start a new transaction.

---

## Query::commit()
`Commits a transaction, i.e. writes all changes since the transaction was started to the database.`

* `Returns`: **true** on success, **false** if the database cannot or will not commit the transaction.

---

## Query::rollback()
`Rolls back a transaction, i.e. reverts all changes since the transaction started.`

* `Returns`: **true** on success, **false** if the database cannot or will not commit the transaction.

