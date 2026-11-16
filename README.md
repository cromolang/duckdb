# DuckDB for Cromo

[DuckDB](https://duckdb.org), the database for analysis that runs inside the program, from a [Cromo](https://cromolang.org) program.

```sh
cromo get github.com/cromolang/duckdb
```

```cromo
import duckdb

i32 main() {
    duckdb.Database data = duckdb.Database("")   // in memory; a path for a file
    duckdb.Connection c = data.connect()
    c.exec("CREATE TABLE numbers AS SELECT range AS n FROM range(1000000)")
    duckdb.Statement sum = c.prepare("SELECT count(*), avg(n) FROM numbers WHERE n % ? = 0")
    sum.setInt(1, 2)
    for (sum.next()) {
        println("{} rows, average {}", sum.getInt(0), sum.getFloat(1))   // 500000 rows, average 499999
    }
    ret 0
}
```

There is nothing else to install. The library of DuckDB is the one that its authors publish: `cromo` downloads it the first time the program is built, checks it by its hash, and puts it next to the executable, `duckdb.dll` on Windows and `libduckdb.so` on Linux. A program is given to others with that file next to it.

## What it has

The names are those of `std.db`, the module of the standard library for SQLite and PostgreSQL, so that a program moves from one to the other with few changes.

- `duckdb.Database(path)` opens the database of a file, made if it is missing, or one in memory with `""`.
- `database.connect()` gives a `Connection`: each thread that works on the database has its own.
- `connection.exec(sql)` runs a statement and gives the rows that it changed; `begin()`, `commit()` and `rollback()` are the transactions.
- `connection.prepare(sql)` gives a `Statement` with parameters, written `?` or `$1`: `setInt`, `setFloat`, `setText`, `setBool` and `setNull` give them, from 1.
- `statement.exec()` runs it; `statement.next()` goes through its rows, whose columns, from 0, are read with `getInt`, `getFloat`, `getText`, `getBool` and `isNull`, and named by `columnName`.
- `duckdb.version()` is the version of DuckDB in use.

DuckDB reads files by itself, from SQL: `SELECT * FROM read_csv('sales.csv')`, `read_parquet('data/*.parquet')`.

What DuckDB refuses, as a statement that is not SQL, is an `Error` with its message. A wrong use of the module, as a column that the row does not have, is a panic.

## Limits of this version

- The values are integers, numbers with a fraction, texts, and true or false: DuckDB converts the other types, as dates and lists, to those, and `getText` gives any value as DuckDB writes it. Bytes (BLOB) are not read yet.
- The rows of a statement are all in memory while it is read.
- Windows and Linux on x86-64. Linux needs glibc 2.29 or later and the C++ library of the system (`libstdc++`).
- A `Statement` must end before its `Connection`, and a `Connection` before its `Database`: declared in that order, as in the example, they do.

## The versions

| This package | DuckDB | Cromo |
|---|---|---|
| v0.1.0 | 1.5.6 | 0.90 or later |

## Developing it

```sh
cromo test duckdb.cro
```

## License

MIT, as DuckDB: see `LICENSE`. DuckDB is a work of the DuckDB Foundation and of its contributors.
