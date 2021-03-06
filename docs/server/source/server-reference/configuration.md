# Configuration Settings

The value of each BigchainDB Server configuration setting is determined according to the following rules:

* If it's set by an environment variable, then use that value
* Otherwise, if it's set in a local config file, then use that value
* Otherwise, use the default value

For convenience, here's a list of all the relevant environment variables (documented below):

`BIGCHAINDB_KEYPAIR_PUBLIC`<br>
`BIGCHAINDB_KEYPAIR_PRIVATE`<br>
`BIGCHAINDB_KEYRING`<br>
`BIGCHAINDB_DATABASE_BACKEND`<br>
`BIGCHAINDB_DATABASE_HOST`<br>
`BIGCHAINDB_DATABASE_PORT`<br>
`BIGCHAINDB_DATABASE_NAME`<br>
`BIGCHAINDB_DATABASE_REPLICASET`<br>
`BIGCHAINDB_SERVER_BIND`<br>
`BIGCHAINDB_SERVER_WORKERS`<br>
`BIGCHAINDB_SERVER_THREADS`<br>
`BIGCHAINDB_CONFIG_PATH`<br>
`BIGCHAINDB_BACKLOG_REASSIGN_DELAY`<br>
`BIGCHAINDB_LOG`<br>
`BIGCHAINDB_LOG_FILE`<br>
`BIGCHAINDB_LOG_LEVEL_CONSOLE`<br>
`BIGCHAINDB_LOG_LEVEL_LOGFILE`<br>
`BIGCHAINDB_LOG_DATEFMT_CONSOLE`<br>
`BIGCHAINDB_LOG_DATEFMT_LOGFILE`<br>
`BIGCHAINDB_LOG_FMT_CONSOLE`<br>
`BIGCHAINDB_LOG_FMT_LOGFILE`<br>
`BIGCHAINDB_LOG_GRANULAR_LEVELS`<br>

The local config file is `$HOME/.bigchaindb` by default (a file which might not even exist), but you can tell BigchainDB to use a different file by using the `-c` command-line option, e.g. `bigchaindb -c path/to/config_file.json start`
or using the `BIGCHAINDB_CONFIG_PATH` environment variable, e.g. `BIGHAINDB_CONFIG_PATH=.my_bigchaindb_config bigchaindb start`.
Note that the `-c` command line option will always take precedence if both the `BIGCHAINDB_CONFIG_PATH` and the `-c` command line option are used.

You can read the current default values in the file [bigchaindb/\_\_init\_\_.py](https://github.com/bigchaindb/bigchaindb/blob/master/bigchaindb/__init__.py). (The link is to the latest version.)

Running `bigchaindb -y configure rethinkdb` will generate a local config file in `$HOME/.bigchaindb` with all the default values, with two exceptions: It will generate a valid private/public keypair, rather than using the default keypair (`None` and `None`).


## keypair.public & keypair.private

The [cryptographic keypair](../appendices/cryptography.html) used by the node. The public key is how the node idenifies itself to the world. The private key is used to generate cryptographic signatures. Anyone with the public key can verify that the signature was generated by whoever had the corresponding private key.

**Example using environment variables**
```text
export BIGCHAINDB_KEYPAIR_PUBLIC=8wHUvvraRo5yEoJAt66UTZaFq9YZ9tFFwcauKPDtjkGw
export BIGCHAINDB_KEYPAIR_PRIVATE=5C5Cknco7YxBRP9AgB1cbUVTL4FAcooxErLygw1DeG2D
```

**Example config file snippet**
```js
"keypair": {
  "public": "8wHUvvraRo5yEoJAt66UTZaFq9YZ9tFFwcauKPDtjkGw",
  "private": "5C5Cknco7YxBRP9AgB1cbUVTL4FAcooxErLygw1DeG2D"
}
```

Internally (i.e. in the Python code), both keys have a default value of `None`, but that's not a valid key. Therefore you can't rely on the defaults for the keypair. If you want to run BigchainDB, you must provide a valid keypair, either in the environment variables or in the local config file. You can generate a local config file with a valid keypair (and default everything else) using `bigchaindb -y configure rethinkdb`.


## keyring

A list of the public keys of all the nodes in the cluster, excluding the public key of this node.

**Example using an environment variable**
```text
export BIGCHAINDB_KEYRING=BnCsre9MPBeQK8QZBFznU2dJJ2GwtvnSMdemCmod2XPB:4cYQHoQrvPiut3Sjs8fVR1BMZZpJjMTC4bsMTt9V71aQ
```

Note how the keys in the list are separated by colons.

**Example config file snippet**
```js
"keyring": ["BnCsre9MPBeQK8QZBFznU2dJJ2GwtvnSMdemCmod2XPB",
            "4cYQHoQrvPiut3Sjs8fVR1BMZZpJjMTC4bsMTt9V71aQ"]
```

**Default value (from a config file)**
```js
"keyring": []
```


## database.backend, database.host, database.port, database.name & database.replicaset

The database backend to use (`rethinkdb` or `mongodb`) and its hostname, port and name. If the database backend is `mongodb`, then there's a fifth setting: the name of the replica set. If the database backend is `rethinkdb`, you *can* set the name of the replica set, but it won't be used for anything.

**Example using environment variables**
```text
export BIGCHAINDB_DATABASE_BACKEND=mongodb
export BIGCHAINDB_DATABASE_HOST=localhost
export BIGCHAINDB_DATABASE_PORT=27017
export BIGCHAINDB_DATABASE_NAME=bigchain
export BIGCHAINDB_DATABASE_REPLICASET=bigchain-rs
```

**Default values**

If (no environment variables were set and there's no local config file), or you used `bigchaindb -y configure rethinkdb` to create a default local config file for a RethinkDB backend, then the defaults will be:
```js
"database": {
    "backend": "rethinkdb",
    "host": "localhost",
    "name": "bigchain",
    "port": 28015
}
```

If you used `bigchaindb -y configure mongodb` to create a default local config file for a MongoDB backend, then the defaults will be:
```js
"database": {
    "backend": "mongodb",
    "host": "localhost",
    "name": "bigchain",
    "port": 27017,
    "replicaset": "bigchain-rs"
}
```


## server.bind, server.workers & server.threads

These settings are for the [Gunicorn HTTP server](http://gunicorn.org/), which is used to serve the [HTTP client-server API](../drivers-clients/http-client-server-api.html).

`server.bind` is where to bind the Gunicorn HTTP server socket. It's a string. It can be any valid value for [Gunicorn's bind setting](http://docs.gunicorn.org/en/stable/settings.html#bind). If you want to allow IPv4 connections from anyone, on port 9984, use '0.0.0.0:9984'. In a production setting, we recommend you use Gunicorn behind a reverse proxy server. If Gunicorn and the reverse proxy are running on the same machine, then use 'localhost:PORT' where PORT is _not_ 9984 (because the reverse proxy needs to listen on port 9984). Maybe use PORT=9983 in that case because we know 9983 isn't used. If Gunicorn and the reverse proxy are running on different machines, then use 'A.B.C.D:9984' where A.B.C.D is the IP address of the reverse proxy. There's [more information about deploying behind a reverse proxy in the Gunicorn documentation](http://docs.gunicorn.org/en/stable/deploy.html). (They call it a proxy.)

`server.workers` is [the number of worker processes](http://docs.gunicorn.org/en/stable/settings.html#workers) for handling requests. If `None` (the default), the value will be (cpu_count * 2 + 1). `server.threads` is [the number of threads-per-worker](http://docs.gunicorn.org/en/stable/settings.html#threads) for handling requests. If `None` (the default), the value will be (cpu_count * 2 + 1). The HTTP server will be able to handle `server.workers` * `server.threads` requests simultaneously.

**Example using environment variables**
```text
export BIGCHAINDB_SERVER_BIND=0.0.0.0:9984
export BIGCHAINDB_SERVER_WORKERS=5
export BIGCHAINDB_SERVER_THREADS=5
```

**Example config file snippet**
```js
"server": {
    "bind": "0.0.0.0:9984",
    "workers": 5,
    "threads": 5
}
```

**Default values (from a config file)**
```js
"server": {
    "bind": "localhost:9984",
    "workers": null,
    "threads": null
}
```

## backlog_reassign_delay

Specifies how long, in seconds, transactions can remain in the backlog before being reassigned.  Long-waiting transactions must be reassigned because the assigned node may no longer be responsive.  The default duration is 120 seconds.

**Example using environment variables**
```text
export BIGCHAINDB_BACKLOG_REASSIGN_DELAY=30
```

**Default value (from a config file)**
```js
"backlog_reassign_delay": 120
```


## log

The `log` key is expected to point to a mapping (set of key/value pairs)
holding the logging configuration.

**Example**:

```
{
    "log": {
        "file": "/var/log/bigchaindb.log",
        "level_console": "info",
        "level_logfile": "info",
        "datefmt_console": "%Y-%m-%d %H:%M:%S",
        "datefmt_logfile": "%Y-%m-%d %H:%M:%S",
        "fmt_console": "%(asctime)s [%(levelname)s] (%(name)s) %(message)s",
        "fmt_logfile": "%(asctime)s [%(levelname)s] (%(name)s) %(message)s",
        "granular_levels": {
            "bichaindb.backend": "info",
            "bichaindb.core": "info"
        }
}
```

**Defaults to**: `"{}"`.

Please note that although the default is `"{}"` as per the configuration file,
internal defaults are used, such that the actual operational default is:

```
{
    "log": {
        "file": "~/bigchaindb.log",
        "level_console": "info",
        "level_logfile": "info",
        "datefmt_console": "%Y-%m-%d %H:%M:%S",
        "datefmt_logfile": "%Y-%m-%d %H:%M:%S",
        "fmt_console": "%(asctime)s [%(levelname)s] (%(name)s) %(message)s",
        "fmt_logfile": "%(asctime)s [%(levelname)s] (%(name)s) %(message)s",
        "granular_levels": {}
}
```

The next subsections explain each field of the `log` configuration.


### log.file
The full path to the file where logs should be written to.

**Example**:

```
{
    "log": {
        "file": "/var/log/bigchaindb/bigchaindb.log"
    }
}
```

**Defaults to**:  `"~/bigchaindb.log"`.

Please note that the user running `bigchaindb` must have write access to the
location.
        

### log.level_console
The log level used to log to the console. Possible allowed values are the ones
defined by [Python](https://docs.python.org/3.6/library/logging.html#levels),
but case insensitive for convenience's sake:

```
"critical", "error", "warning", "info", "debug", "notset"
```

**Example**:

```
{
    "log": {
        "level_console": "info"
    }
}
```

**Defaults to**: `"info"`.


### log.level_logfile
The log level used to log to the log file. Possible allowed values are the ones
defined by [Python](https://docs.python.org/3.6/library/logging.html#levels),
but case insensitive for convenience's sake:

```
"critical", "error", "warning", "info", "debug", "notset"
```

**Example**:

```
{
    "log": {
        "level_file": "info"
    }
}
```

**Defaults to**: `"info"`.


### log.datefmt_console
The format string for the date/time portion of a message, when logged to the
console.

**Example**:

```
{
    "log": {
        "datefmt_console": "%x %X %Z"
    }
}
```

**Defaults to**: `"%Y-%m-%d %H:%M:%S"`.

For more information on how to construct the format string please consult the
table under Python's documentation of
 [`time.strftime(format[, t])`](https://docs.python.org/3.6/library/time.html#time.strftime)

### log.datefmt_logfile
The format string for the date/time portion of a message, when logged to a log
 file.

**Example**:

```
{
    "log": {
        "datefmt_logfile": "%c %z"
    }
}
```

**Defaults to**: `"%Y-%m-%d %H:%M:%S"`.

For more information on how to construct the format string please consult the
table under Python's documentation of
 [`time.strftime(format[, t])`](https://docs.python.org/3.6/library/time.html#time.strftime)


### log.fmt_console
A string used to format the log messages when logged to the console.

**Example**:

```
{
    "log": {
        "fmt_console": "%(asctime)s [%(levelname)s] %(message)s %(process)d"
    }
}
```

**Defaults to**: `"[%(asctime)s] [%(levelname)s] (%(name)s) %(message)s (%(processName)-10s - pid: %(process)d)"`

For more information on possible formatting options please consult Python's
documentation on
[LogRecord attributes](https://docs.python.org/3.6/library/logging.html#logrecord-attributes)


### log.fmt_logfile
A string used to format the log messages when logged to a log file.

**Example**:

```
{
    "log": {
        "fmt_logfile": "%(asctime)s [%(levelname)s] %(message)s %(process)d"
    }
}
```

**Defaults to**: `"[%(asctime)s] [%(levelname)s] (%(name)s) %(message)s (%(processName)-10s - pid: %(process)d)"`

For more information on possible formatting options please consult Python's
documentation on
[LogRecord attributes](https://docs.python.org/3.6/library/logging.html#logrecord-attributes)


### log.granular_levels
Log levels for BigchainDB's modules. This can be useful to control the log
level of specific parts of the application. As an example, if you wanted the
logging of the `core.py` module to be more verbose, you would set the
 configuration shown in the example below.

**Example**:

```
{
    "log": {
        "granular_levels": {
            "bichaindb.core": "debug"
        }
}
```

**Defaults to**: `"{}"`
