# Filesystem example

## Summary

This is a fuse filesystem using cockroach as a backing store.
The implemented features attempt to be posix compliant.
See `main.go` for more details, including implemented features and caveats.

## Running

Run against an existing cockroach node or cluster.

#### Development node
```
# Build cockroach binary from https://github.com/cockroachdb/cockroach
# Start it in insecure mode (listens on localhost:15432)
./cockroach start --insecure

# Build filesystem example.
# Start it with:
mkdir /tmp/foo
./filesystem "postgresql://admin@localhost:26257?sslmode=disable" /tmp/foo
# <CTRL-C> to umount and quit
# Use /tmp/foo as a filesystem.
```

#### Insecure node or cluster
```
# Launch your node or cluster in insecure mode (with --insecure passed to cockroach).
# Find a reachable address: [mycockroach:15432].
# Run the example with:
mkdir /tmp/foo
./filesystem postgresql://root@mycockroach:15432/?sslmode=disable /tmp/foo
# <CTRL-C> to umount and quit
# Use /tmp/foo as a filesystem.
```

#### Secure node or cluster
```
# Launch your node or cluster in secure mode with certificates in [mycertsdir]
# Find a reachable address:[mycockroach:15432].
# Run the example with:
mkdir /tmp/foo
./filesystem postgresqls://root@mycockroach:15432/?certs=verify-ca&sslcert=mycertsdir/root.client.crt&sslkey=mycertsdir/root.client.key&sslrootcert=mycertsdir/ca.crt /tmp/foo
# <CTRL-C> to umount and quit
# Use /tmp/foo as a filesystem.
```

## Schema

Schema and sample data for a file moved to `/tmp/foo`.

```
root@:26257/fs> \d;            
  schema_name | table_name | type  | owner | estimated_row_count | locality
--------------+------------+-------+-------+---------------------+-----------
  public      | block      | table | admin |                   0 | NULL
  public      | inode      | table | admin |                   0 | NULL
  public      | namespace  | table | admin |                   0 | NULL
(3 rows)

root@:26257/fs> \d namespace; 
  column_name | data_type | is_nullable | column_default | generation_expression |     indices      | is_hidden
--------------+-----------+-------------+----------------+-----------------------+------------------+------------
  parentid    | INT8      |    false    | NULL           |                       | {namespace_pkey} |   false
  name        | STRING    |    false    | NULL           |                       | {namespace_pkey} |   false
  id          | INT8      |    true     | NULL           |                       | {namespace_pkey} |   false
(3 rows)

root@:26257/fs> SELECT * FROM namespace;
  parentid |          name          |         id
-----------+------------------------+---------------------
         1 | eBPF_Observability.pdf | 777345548497321985
(1 row)

root@:26257/fs> \d inode;                                                                                                                                                                                                                                                                           
  column_name | data_type | is_nullable | column_default | generation_expression |   indices    | is_hidden
--------------+-----------+-------------+----------------+-----------------------+--------------+------------
  id          | INT8      |    false    | NULL           |                       | {inode_pkey} |   false
  inode       | STRING    |    true     | NULL           |                       | {inode_pkey} |   false
(2 rows)

root@:26257/fs> SELECT * FROM inode;    
          id         |                                 inode
---------------------+-------------------------------------------------------------------------
  777345548497321985 | {"ID":777345548497321985,"Mode":493,"SymlinkTarget":"","Size":1134878}
(1 row)

root@:26257/fs> \d block;           
  column_name | data_type | is_nullable | column_default | generation_expression |   indices    | is_hidden
--------------+-----------+-------------+----------------+-----------------------+--------------+------------
  id          | INT8      |    false    | NULL           |                       | {block_pkey} |   false
  block       | INT8      |    false    | NULL           |                       | {block_pkey} |   false
  data        | BYTES     |    true     | NULL           |                       | {block_pkey} |   false
(3 rows)

root@:26257/fs> SELECT id,block FROM block;
          id         | block
---------------------+--------
  777345548497321985 |     0
  777345548497321985 |     1
  777345548497321985 |     2
  ...
  777345548497321985 |   277
(278 rows)

```