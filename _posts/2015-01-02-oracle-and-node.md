---
title: Oracle and Node
summary: Oracle Database 11g Enterprise Edition Release 11.2.0.3.0 – 64bit Production access by Node v0.10.33 via npm module `oracle` v0.3.7
---

Adventures getting Node to connect to an Oracle database.

The good news? Someone has done the hard parts of [connecting to Oracle’s drivers](https://www.npmjs.com/package/oracle) (thank you!).

The bad news? I’m getting segmentation faults.

Right, so first we need to [know what version of Oracle DB](https://community.oracle.com/thread/2250946?tstart=0) we’re connecting to. In Oracle’s SQL Developer

{% highlight SQL %}
SELECT * FROM V$VERSION
{% endhighlight %}
tells me more, but `11.2.0.3.0` is the relevant bit.

So then we can follow the instructions to [install the npm oracle package](https://github.com/joeferner/node-oracle/blob/master/INSTALL.md)


I’m on Ubuntu 14.04.1 ×64

I found that trying to compile with the non-x64 downloads ended in failure. Instead of the download page linked to by `INSTALL.md`, I found my 11.2.0.3.0 downloads at [http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html). Specifically, the **basiclight** and **sdk** downloads.

I was just trying to use this on an experimental program, so instead of a system accessible location I downloaded to `./instantclient/` and the subsequent `unzip`ping placed the contents into `./instantclient/instantclient_11_2`


From there, rather than modify system variables (again, just an experiment project) I created two files, `install.sh` and `start.sh`.

install.sh:

{% highlight bash %}
#!/bin/bash
PROJECT_HOME=`pwd`

echo "PROJECT_HOME: $PROJECT_HOME"

# due to the use of Oracle Instant client, we need some system variables
# see https://github.com/joeferner/node-oracle/blob/master/INSTALL.md
export OCI_HOME=$PROJECT_HOME/instantclient/instantclient_11_2
export OCI_LIB_DIR=$OCI_HOME
export OCI_INCLUDE_DIR=$OCI_HOME/sdk/include
export OCI_VERSION=11
export NLS_LANG=AMERICAN_AMERICA.UTF8

echo "OCI_HOME: $OCI_HOME"

cd $OCI_LIB_DIR
ln -s libclntsh.so.11.1 libclntsh.so
ln -s libocci.so.11.1 libocci.so

sudo apt-get install libaio1
#sudo yum install libaio

echo $OCI_HOME | sudo tee -a /etc/ld.so.conf.d/oracle_instant_client.conf
sudo ldconfig

cd $PROJECT_HOME
# already added oracle to package.json
npm install
# or use
#npm install oracle
{% endhighlight %}

and start.sh

{% highlight bash %}
#!/bin/bash

# due to the use of Oracle Instant client, we need some system variables
# see https://github.com/joeferner/node-oracle/blob/master/INSTALL.md
export OCI_HOME=$HERE/instantclient/instantclient_11_2
export OCI_LIB_DIR=$OCI_HOME
export OCI_INCLUDE_DIR=$OCI_HOME/sdk/include
export OCI_VERSION=11
export NLS_LANG=AMERICAN_AMERICA.UTF8

#node index.js
node oracleTest.js
{% endhighlight %}

As you can guess, I’m testing the connection in `oracleTest.js`

{% highlight javascript %}
var oracle = require('oracle');

var connString = "use your own";
var connectData = { "tns": connString, "user": "pixnbits", "password": "incorrect" };
// http://unijokes.com/joke-4826/

oracle.connect(connectData, function(err, connection) {
    if (err) { console.log("Error connecting to db:", err); return; }

    connection.execute("SELECT systimestamp FROM dual", [], function(err, results) {
        if (err) { console.log("Error executing query:", err); return; }

        console.log(results);
        connection.close(); // call only when query is finished executing
    });

});
{% endhighlight %}

so then

{% highlight bash %}
pixnbits@silence:~/experimental-project$ ./start.sh 
[ { SYSTIMESTAMP: Fri Jan 02 2015 08:29:15 GMT-0700 (MST) } ]
{% endhighlight %}

Looks good! However, a simple `SELECT` is not so sunny

{% highlight javascript %}
var oracle = require('oracle');

var connString = "use your own";
var connectData = { "tns": connString, "user": "pixnbits", "password": "incorrect" };
// http://unijokes.com/joke-4826/

oracle.connect(connectData, function(err, connection) {
    if (err) { console.log("Error connecting to db:", err); return; }

    connection.setPrefetchRowCount(50);
    var reader = connection.reader("SELECT * FROM table_with_data WHERE ROWNUM &lt; 10", []);

    function doRead(cb) {
        reader.nextRow(function(err, row) {
            if (err) return cb(err);
            if (row) {
                // do something with row
                console.log("row",row);
                // recurse to read next record
                return doRead(cb)
            } else {
                // we are done
                return cb();
            }
        })
    }

    doRead(function(err) {
        if (err) throw err; // or log it
        console.log("all records processed");
    });

});
{% endhighlight %}

{% highlight bash %}
pixnbits@silence:~/experimental-project$ ./start.sh 
row { 
  ...data...
 }
./start.sh: line 12: 11874 Segmentation fault      (core dumped) node oracleTest.js
{% endhighlight %}

Sometimes it’s an illegal instruction.


[EDIT]


Why? Well…db connections 101: always close the connection when you’re done.

So, either

{% highlight javascript %}
    function doRead(cb) {
        reader.nextRow(function(err, row) {
            if (err) return cb(err);
            if (row) {
                // do something with row
                console.log("row",row);
                // recurse to read next record
                return doRead(cb)
            } else {
                // we are done
                connection.close();
                return cb();
            }
        })
    }
{% endhighlight %}

or

{% highlight javascript %}
oracle.connect(connectData, function(err, connection) {
    if (err) { console.log("Error connecting to db:", err); return; }

    // http://nodejs.org/api/process.html#process_event_exit
    process.on('exit', function(){
        console.log('process exit, close db connection');
        connection.close();
    });
{% endhighlight %}

I might add a note in the README as a pull request, just for people like me who forgot.


**Obviously this is me using software I don’t own or control. Obviously I’m not endorsing or making any other recommendations of any of them. Those vendors may contact me if they want legalese here.**
