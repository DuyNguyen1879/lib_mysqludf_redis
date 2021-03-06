lib_mysqludf_redis
==================
Provides UDF commands to access Redis from Mysql/MariaDB.

[English](README.md) | [繁體中文](README.hant.md) | [日本語](README.jp.md)


#### Table of Contents
* [Synopsis](#synopsis)
* [System Requirements](#system-requirements)
* [Compilation and Install Plugin Library](#compilation-and-install-plugin-library)
    * [Compilation Argumnets](#compilation-argumnets)
    * [Compilation Variable](#compilation-variable)
* [Install and Uninstall UDF](#install-and-uninstall-udf)
* [Usage](#usage)
* [TODO](#todo)
* [Copyright and License](#copyright-and-license)
* [See Also](#see-also)


Synopsis
--------
![Alt text](https://g.gravizo.com/source/figure01?https%3A%2F%2Fraw.githubusercontent.com%2FIdeonella-sakaiensis%2Flib_mysqludf_redis%2Fmaster%2FREADME.md?3)
<details>
<summary></summary>
figure01
  digraph G {

    rankdir = "LR";
    size ="8,8";
    edge [
        fontname = "Consolas"
        fontsize = 10
    ];
    MariaDB [
        label = "MariaDB\n(presistence)"
        shape = "box"
    ];
    Redis [
        label = "Redis\n(cached)"
        shape = "box"
    ];
    edge [
        fontcolor = "blue"
        color = "blue"
    ];
    writer;
    writer:e -> MariaDB [
        label="INSERT\nUPDATE\nDELETE"
    ];
    MariaDB -> Redis [
        label = "SET"
    ];
    edge [
        fontcolor = "red"
        color = "red"
    ];
    reader;
    reader:e -> MariaDB [
        label="SELECT"
    ];
    MariaDB -> Redis [
        label = "GET"
    ];
    edge [
        fontcolor = "default"
        color = "default"
        dir ="none"
        arrowhead="none"
        arrowtail="none"
        penwidth = 0.5
        style="dashed"
    ];
    node [
        fontname = "Consolas"
        fontsize = 10
        penwidth = 0.5
        color    = "gray"
        shape = "record"
        style = "rounded"
    ];
    MariaDB_Data [
      label = <<TABLE border="0" cellspacing="0" cellborder="1"><TR><TD COLSPAN="2">MariaDB data</TD></TR><TR><TD>item</TD><TD>qty</TD></TR><TR><TD>shoes</TD><TD>35</TD></TR><TR><TD>books</TD><TD>158</TD></TR></TABLE>>
    ];
    {
      rank = "same";
      MariaDB:n -> MariaDB_Data:s;
    }
    Redis_Data [
      label = <<TABLE border="0" cellspacing="0" cellborder="1"><TR><TD COLSPAN="2">Redis data</TD></TR><TR><TD>item</TD><TD>qty</TD></TR><TR><TD>shoes</TD><TD>35</TD></TR><TR><TD>books</TD><TD>158</TD></TR></TABLE>>
    ];
    {
      rank = "same";
      Redis:n -> Redis_Data:s;
    }
  }
figure01
</details>


[Back to TOC](#table-of-contents)


System Requirements
-------------------
* Architectures: Linux 64-bit(x64)
* Compilers: GCC 4.1.2+
* MariaDB 5.5+
* Redis 1.2+
* Dependencies:
    * MariaDB development library 5.5+
    * hiredis 0.13.3+
    * cJSON 1.6+

[Back to TOC](#table-of-contents)


Compilation and Install Plugin Library
--------------------------------------
Installing compilation tools
> CentOS
> ```bash
> # install tools
> $ yum install -y make wget gcc git
>
> # install mariadb development tool
> $ yum install -y mariadb-devel
> ```

> Debian
> ```bash
> # install tools
> $ apt-get install -y make wget gcc git
>
> # install mariadb development tool
> $ apt-get install -y libmariadb-dev
> ```

> FreeBSD
> ```bash
> # install tools
> $ pkg install -y gmake wget gcc git-lite
> ```

To compile the plugin library just simply type `make` and `make install`. -or- `gmake` and `gmake install` on FreeBSD.
```bash
$ make

# install plugin library to plugin directory
$ make install

# install UDF to Mysql/MariaDB server
$ make installdb
```
> **NOTE**: If the Mysql/MariaDB is an earlier version or installed from source code, the default include path might be invalid; use `` make INCLUDE_PATH=`mysql_config --variable=pkgincludedir` `` to assign `INCLUDE_PATH` variable for compilation.


#### Compilation Argumnets
* `install`

  Install the plugin library to Mysql plugin directory.

* `installdb`

  Install UDFs to Mysql/MariaDB server.

* `uninstalldb`

  Uninstall UDFs from Mysql/MariaDB server.

* `clean`

  Clear the compiled files and resources.

* `distclean`

  Like the `clean` and also remove the dependencies resources.

#### Compilation Variable
The compilation variable can be use in `make`:
* `HIREDIS_MODULE_VER`

  The [hiredis](https://github.com/redis/hiredis) version to be compiled. If it is not specified, the default value is `0.13.3`.

* `CJSON_MODULE_VER`

  The [cJSON](https://github.com/DaveGamble/cJSON) version to be compiled. If it is not specified, the default value is `1.6.0`

* `INCLUDE_PATH`

  The MariaDB or Mysql C header path. If it is not specified, the default will be the Mysql variable `pkgincludedir`. The value can be displayed by executing the following command:
  ``` bash
  $ echo `mysql_config --variable=pkgincludedir`/server
  ```

* `PLUGIN_PATH`

  The MariaDB or Mysql plugin path. The value can be obtained via running the sql statement `SHOW VARIABLES LIKE '%plugin_dir%';` in MariaDB/Mysql server. If it is not specified, the default will be Mysql variable `plugindir`. The value can be displayed by executing the following command:
  ``` bash
  $ mysql_config --plugindir
  ```

example:
```bash
# specify the plugin install path with /opt/mysql/plugin
$ make PLUGIN_PATH=/opt/mysql/plugin
$ make install
```
[Back to TOC](#table-of-contents)


Install and Uninstall UDF
-------------------------
To install UDF from `make`:

```bash
$ make installdb
```

> or executing the following sql statement on Mysql/MariaDB server:
> 
> ```sql
> mysql>  CREATE FUNCTION `redis` RETURNS STRING SONAME 'lib_mysqludf_redis.so';
> ```

To uninstall UDF with `make uninstalldb` -or- executing the following sql statement on Mysql/MariaDB server:

```sql
mysql>  DROP FUNCTION IF EXISTS `redis`;
```

[Back to TOC](#table-of-contents)


Usage
-----
### `redis`(*$connection_string*,  *$command*,  [*$args...*])

Call a Redis command by specified `$connection_string`, `$command`, and individual arguments.

* **$connection_string** - represent the Redis server to be connected, the value is a DSN connection string, must be one of following type:
  - **redis**://:_`<password>`_**@**_`<host>`_:_`<port>`_**/**_`<database>`_**/**
  - **redis**://:_`<password>`_**@**_`<host>`_**/**_`<database>`_**/**
  - **redis**://**@**_`<host>`_:_`<port>`_**/**_`<database>`_**/**
  - **redis**://**@**_`<host>`_**/**_`<database>`_**/**
* **$command**, **$args...** - the Redis command and arguments. See also [https://redis.io/commands](https://redis.io/commands) for further details.


The function returns a JSON string indicating success or failure of the operation.
> the success output:
> ```json
> {
>    "out": "OK"
> }
> ```

> the failure output:
> ```json
> {
>    "err": "Connection refused"
> }
> ```

The following examples illustrate how to use the function contrast with `redis-cli` utility.
```sql
/*
  the following statement likes:

    $ redis-cli -h 127.0.0.1 -n 8 PING
    PONG
*/
mysql>  SELECT `redis`('redis://@127.0.0.1/8/', 'PING')\G
*************************** 1. row ***************************
`redis`('redis://@127.0.0.1/8/', 'PING'): {
        "out":  "PONG"
}
1 row in set (0.00 sec)



/*
  the following statement likes:

    $ redis-cli -h 127.0.0.1 -a foobared -n 8 PING
    PONG
*/
mysql>  SELECT `redis`('redis://:foobared@127.0.0.1/8/', 'PING')\G
*************************** 1. row ***************************
`redis`('redis://:foobared@127.0.0.1/8/', 'PING'): {
        "out":  "PONG"
}
1 row in set (0.00 sec)



/*
    $ redis-cli -h 127.0.0.1 -p 80 -n 8 PING
    Could not connect to Redis at 127.0.0.1:80: Connection refused
*/
mysql>  SELECT `redis`('redis://@127.0.0.1:80/8/', 'PING')\G
*************************** 1. row ***************************
`redis`('redis://@127.0.0.1:80/8/', 'PING'): {
        "err":  "Connection refused"
}
1 row in set (0.00 sec)



/*
    $ redis-cli -h 127.0.0.1 -n 8 HMSET myhash field1 Hello field2 World
    OK
*/
mysql>  SELECT `redis`('redis://@127.0.0.1/8/', 'HMSET', 'myhash', 'field1', 'Hello', 'field2', 'World')\G
*************************** 1. row ***************************
`redis`('redis://@127.0.0.1/8/', 'HMSET', 'myhash', 'field1', 'Hello', 'field2', 'World'): {
        "out":  "OK"
}
1 row in set (0.00 sec)



/*
    $ redis-cli -h 127.0.0.1 -n 8 HGET myhash field1
    "Hello"
*/
mysql>  SELECT `redis`('redis://@127.0.0.1/8/', 'HGET', 'myhash', 'field1')\G
*************************** 1. row ***************************
`redis`('redis://@127.0.0.1/8/', 'HGET', 'myhash', 'field1'): {
        "out":  "Hello"
}
1 row in set (0.00 sec)



-- redis-cli -h 127.0.0.1 -n 0 SET foo bar
mysql>  SELECT `redis`('redis://@127.0.0.1/0/', 'SET', 'foo', 'bar')

-- redis-cli -h 127.0.0.1 -n 0 SCAN 0 MATCH prefix*
mysql>  SELECT `redis`('redis://@127.0.0.1/0/', 'SCAN', '0', 'MATCH', 'prefix*')
```

[Back to TOC](#table-of-contents)


TODO
----
- [x] implement Redis Authentication (2017-12-30)
- [ ] add redis DSN string builder function

[Back to TOC](#table-of-contents)


Copyright and License
---------------------
See [LICENSE](LICENSE) for further details.

[Back to TOC](#table-of-contents)


See Also
---------

[Back to TOC](#table-of-contents)
