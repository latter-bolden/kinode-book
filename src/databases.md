# Databases

The runtime currently comes preloaded with 2 databases, a key value (RocksDB) and sqlite.

These can be created, accessed, and shared amongst processes.
The APIs for doing so you can find here: [KV](./apis/kv.md) and [SQLite](./apis/sqlite.md).

Similarly to files in the VFS, they are accessed by `package_id` and a `db` name.
Capabilities to read and write can be shared with other processes; processes within a given package have access by default.

All examples are using the [nectar_process_lib](https://github.com/uqbar-dao/process_lib) functions.

## Usage

#### KV

```rust
// opens or creates a kv db named birthdays in our package.
let kv = kv::open(our.package_id(), "birthdays")?;

kv.set(b"tacitus".to_vec(), b"53 CE".to_vec(), None)?;

let bday = kv.get(b"tacitus".to_vec())?;

println!("got a bday: {}", String::from_utf8(bday)?);
```

#### SQLite

```rust
// opens or creates sqlite db named users in our package.
let db = sqlite::open(our.package_id(), "users")?;

let create_table_statement =
    "CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT NOT NULL);".to_string();

db.write(create_table_statement, vec![], None)?;

let insert_statement = "INSERT INTO users (name) VALUES (?), (?), (?);".to_string();
let params = vec![
    serde_json::Value::String("Bob".to_string()),
    serde_json::Value::String("Charlie".to_string()),
    serde_json::Value::String("Dave".to_string()),
];

sqlite.write(insert_statement, params, None)?;

let select_statement = "SELECT * FROM users;".to_string();
let rows = sqlite.read(select_statement, vec![])?;
// rows: Vec<HashMap<String, serde_json::Value>>
println!("rows: {}", rows.len());
```

## References

- [KV API](./apis/kv.md)
- [SQLite API](./apis/sqlite.md)
- [RocksDB](https://github.com/rust-rocksdb/rust-rocksdb)
- [SQLite](https://www.sqlite.org/docs.html)
- [nectar_process_lib](https://github.com/uqbar-dao/process_lib)
