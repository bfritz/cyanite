cyanite: cassandra backed metric storage web service
====================================================

![cyanite](http://upload.wikimedia.org/wikipedia/commons/thumb/0/00/Kyanite_crystals.jpg/320px-Kyanite_crystals.jpg)

Cyanite is a metric storage daemon, exposing both
a carbon listener and a simple web service. Its aim is
to become a simple, scalable and drop-in replacement for
graphite's backend.

Graphite is a powerful graphing solution. It sports a somewhat aged but
very powerful web interface. Carbon is graphite's storage daemon,
written in python which writes out whisper or ceres file.

## Compiling

Cyanite is a clojure application and thus can be built as a standalone JAR file.
Building cyanite needs a working [leiningen](http://leiningen.org) installation,
as well as a java JRE and JDK. Once the prerequisites are met run the following:

```
lein uberjar
```

The resulting artifact will be stored in `target/cyanite-0.1.0-standalone.jar`

## Runtime dependencies

You will need a running cassandra cluster, the simplest way to get up and running
is to follow the instructions as available here (using the `20x` branch):
http://wiki.apache.org/cassandra/DebianPackaging.

You will need a cassandra keyspace (rough equivalent of an SQL database) with the
following schema (also available in doc/schema.cql):

```
CREATE KEYSPACE metric WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': '1'
};

USE metric;

CREATE TABLE metric (
  period int,
  rollup int,
  path text,
  time bigint,
  data list<double>,
  PRIMARY KEY ((period, rollup, path), time)
) WITH
  bloom_filter_fp_chance=0.010000 AND
  caching='KEYS_ONLY' AND
  comment='' AND
  dclocal_read_repair_chance=0.000000 AND
  gc_grace_seconds=864000 AND
  index_interval=128 AND
  read_repair_chance=0.100000 AND
  replicate_on_write='true' AND
  populate_io_cache_on_flush='false' AND
  default_time_to_live=0 AND
  speculative_retry='NONE' AND
  memtable_flush_period_in_ms=0 AND
  compaction={'class': 'SizeTieredCompactionStrategy'} AND
  compression={'sstable_compression': 'LZ4Compressor'};

```

You can create this keyspace by running the following command:

```
cqlsh < doc/schema.cql
```

## Configuring

Cyanite is configured from a `YAML` file. The default path for this
path is `/etc/cyanite.yaml` though a different path can be provided
on the command line.

```yaml
carbon:
  host: "127.0.0.1"
  port: 2003
  rollups:
    - period: 60480
      rollup: 10
    - period: 105120
      rollup: 600
http:
  host: "127.0.0.1"
  port: 8080
logging:
  level: info
  console: true
  files:
    - "/tmp/cyanite.log"
store:
  cluster: 'localhost'
  keyspace: 'metric'
```

## Running

```
 Switches                 Default  Desc
 --------                 -------  ----
 -h, --no-help, --help    false    Show help
 -f, --path                        Configuration file path
 -q, --no-quiet, --quiet  false    Suppress output
```

## Wiring up graphite-web to cyanite

To display cyanite graphs from grahite, you need to run graphite-web
from this branch for now: https://github.com/brutasse/graphite-web/tree/exoscale-graphite, you
will also need the following module installed: https://github.com/brutasse/graphite-cyanite.
Stay tuned for a seamless installation procedure.

## Thanks

Thanks go out to @brutasse for his work on https://github.com/brutasse/graphite-cyanite and modifications
in his `graphite-exoscale` branch of graphite-web: https://github.com/brutasse/graphite-web.

We're also indebted to the creator of graphite which we've been using for a while
and for the rest of the crew at exoscale.

## License

Copyright 2013 Pierre-Yves Ritschard <pyr@spootnik.org>

Permission to use, copy, modify, and distribute this software for any
purpose with or without fee is hereby granted, provided that the above
notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
