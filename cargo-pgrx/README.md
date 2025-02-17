# cargo-pgrx

`cargo-pgrx` is a Cargo subcommand for managing `pgrx`-based Postgres extensions.

You'll want to use `cargo pgrx` during your extension development process. It automates the process of creating new Rust crate projects, auto-generating the SQL schema for your extension, installing your extension locally for testing with Postgres, and running your test suite against one or more versions of Postgres.

A video walkthrough of its abilities can be found here: https://www.twitch.tv/videos/684087991

## Installing

Install via crates.io:

```console
$ cargo install --locked cargo-pgrx
```

As new versions of `pgrx` are released, you'll want to make sure you run this command again to update it. You should also reinstall `cargo-pgrx` whenever you update `rustc` so that the same compiler is used to build `cargo-pgrx` and your Postgres extensions. You can force `cargo` to reinstall an existing crate by passing `--force`.

## Usage

```console
$ cargo pgrx --help
Cargo subcommand for 'pgrx' to make Postgres extension development easy

Usage: cargo pgrx [OPTIONS] <COMMAND>

Commands:
  init          Initialize pgrx development environment for the first time
  info          Provides information about pgrx-managed development environment
  start         Start a pgrx-managed Postgres instance
  stop          Stop a pgrx-managed Postgres instance
  status        Is a pgrx-managed Postgres instance running?
  new           Create a new extension crate
  install       Install the extension from the current crate to the Postgres specified by whatever `pg_config` is currently on your $PATH
  sudo-install  Like `cargo pgrx install`, but uses `sudo` to copy the extension files
  package       Create an installation package directory
  schema        Generate extension schema files
  run           Compile/install extension to a pgrx-managed Postgres instance and start psql
  connect       Connect, via psql, to a Postgres instance
  test          Run the test suite for this crate
  get           Get a property from the extension control file
  cross         Cargo subcommand for 'pgrx' to make Postgres extension development easy
  help          Print this message or the help of the given subcommand(s)

Options:
  -v, --verbose...  Enable info logs, -vv for debug, -vvv for trace
  -h, --help        Print help
  -V, --version     Print version
```

## Environment Variables

- `PGRX_HOME` - If set, overrides `pgrx`'s default directory of `~/.pgrx/`
- `PGRX_BUILD_FLAGS` - If set during `cargo pgrx run/test/install`, these additional flags are passed to `cargo build` while building the extension
- `PGRX_BUILD_VERBOSE` - Set to true to enable verbose "build.rs" output -- useful for debugging build issues
- `HTTPS_PROXY` - If set during `cargo pgrx init`, it will download the Postgres sources using these proxy settings. For more details refer to the [env_proxy crate documentation](https://docs.rs/env_proxy/*/env_proxy/fn.for_url.html).
- `PGRX_IGNORE_RUST_VERSIONS` - Set to true to disable the `rustc` version check we have when performing schema generation (schema generation requires the same version of `rustc` be used to build `cargo-pgrx` as the crate in question).

## First Time Initialization

```console
$ cargo pgrx init
   Discovered Postgres v12.16, v13.12, v14.9, v15.4, v16.0
  Downloading Postgres v14.9 from https://ftp.postgresql.org/pub/source/v14.9/postgresql-14.9.tar.bz2
  Downloading Postgres v15.4 from https://ftp.postgresql.org/pub/source/v15.4/postgresql-15.4.tar.bz2
  Downloading Postgres v12.16 from https://ftp.postgresql.org/pub/source/v12.16/postgresql-12.16.tar.bz2
  Downloading Postgres v13.12 from https://ftp.postgresql.org/pub/source/v13.12/postgresql-13.12.tar.bz2
  Downloading Postgres v16.0 from https://ftp.postgresql.org/pub/source/v16.0/postgresql-16.0.tar.bz2
     Removing /home/you/.pgrx/12.16_unpack
     Removing /home/you/.pgrx/14.9_unpack
    Untarring Postgres v12.16 to /home/you/.pgrx/12.16_unpack
    Untarring Postgres v14.9 to /home/you/.pgrx/14.9_unpack
     Removing /home/you/.pgrx/15.4_unpack
    Untarring Postgres v15.4 to /home/you/.pgrx/15.4_unpack
     Removing /home/you/.pgrx/16.0_unpack
    Untarring Postgres v16.0 to /home/you/.pgrx/16.0_unpack
     Removing /home/you/.pgrx/13.12_unpack
    Untarring Postgres v13.12 to /home/you/.pgrx/13.12_unpack
     Removing /home/you/.pgrx/12.16
     Removing /home/you/.pgrx/14.9
     Renaming /home/you/.pgrx/12.16_unpack/postgresql-12.16 -> /home/you/.pgrx/12.16
  Configuring Postgres v12.16
     Renaming /home/you/.pgrx/14.9_unpack/postgresql-14.9 -> /home/you/.pgrx/14.9
  Configuring Postgres v14.9
     Removing /home/you/.pgrx/15.4
     Renaming /home/you/.pgrx/15.4_unpack/postgresql-15.4 -> /home/you/.pgrx/15.4
  Configuring Postgres v15.4
     Removing /home/you/.pgrx/16.0
     Renaming /home/you/.pgrx/16.0_unpack/postgresql-16.0 -> /home/you/.pgrx/16.0
  Configuring Postgres v16.0
     Removing /home/you/.pgrx/13.12
     Renaming /home/you/.pgrx/13.12_unpack/postgresql-13.12 -> /home/you/.pgrx/13.12
  Configuring Postgres v13.12
    Compiling Postgres v16.0
    Compiling Postgres v12.16
    Compiling Postgres v14.9
    Compiling Postgres v15.4
    Compiling Postgres v13.12
   Installing Postgres v12.16 to /home/you/.pgrx/12.16/pgrx-install
   Installing Postgres v13.12 to /home/you/.pgrx/13.12/pgrx-install
   Installing Postgres v14.9 to /home/you/.pgrx/14.9/pgrx-install
   Installing Postgres v15.4 to /home/you/.pgrx/15.4/pgrx-install
   Installing Postgres v16.0 to /home/you/.pgrx/16.0/pgrx-install
   Validating /home/you/.pgrx/12.16/pgrx-install/bin/pg_config
   Validating /home/you/.pgrx/13.12/pgrx-install/bin/pg_config
   Validating /home/you/.pgrx/14.9/pgrx-install/bin/pg_config
   Validating /home/you/.pgrx/15.4/pgrx-install/bin/pg_config
   Validating /home/you/.pgrx/16.0/pgrx-install/bin/pg_config
```

`cargo pgrx init` is required to be run once to properly configure the `pgrx` development environment.

As shown by the screenshot above, it downloads the latest releases of supported Postgres versions, configures them, compiles them, and installs them to `~/.pgrx/`, including all [`contrib`](https://www.postgresql.org/docs/current/contrib.html) extensions and tools included with Postgres. Other `pgrx` commands such as `run` and `test` will fully manage and otherwise use these Postgres installations for you.

`pgrx` is designed to support multiple Postgres versions in such a way that during development, you'll know if you're trying to use a Postgres API that isn't common across all versions. It's also designed to make testing your extension against these versions easy. This is why it requires you to have all fully compiled and installed versions of Postgres during development.

In cases when default ports pgrx uses to run PostgreSQL within are not available, one can specify
custom values for these during initialization using `--base-port` and `--base-testing-port`
options. One of the use cases for this is using multiple installations of pgrx (using `$PGRX_HOME` variable)
when developing multiple extensions at the same time. These values can be later changed in `$PGRX_HOME/config.toml`.

If you want to use your operating system's package manager to install Postgres, `cargo pgrx init` has optional arguments that allow you to specify where they're installed (see below).

What you're telling `cargo pgrx init` is the full path to `pg_config` for each version.

For any version you specify, `cargo pgrx init` will forego downloading/compiling/installing it. `pgrx` will then use that locally-installed version just as it uses any version it downloads/compiles/installs itself.

However, if the "path to pg_config" is the literal string `download`, then `pgrx` will download and compile that version of Postgres for you.

When the various `--pgXX` options are specified, these are the **only** versions of Postgres that `pgrx` will manage for you.

You'll also want to make sure you have the "postgresql-server-dev" package installed for each version you want to manage yourself. If you need to customize the configuration of the Postgres build, you can use `--configure-flag` to pass optins to the `configure` script. For example, you could use `--configure-flag=--with-ssl=openssl` to enable SSL support or `--configure-flag=--with-libraries=/path/to/libs` to use a non-standard location for dependency libraries. This flag can be used multiple times to pass multiple configuration options.

Once complete, `cargo pgrx init` also creates a configuration file (`~/.pgrx/config.toml`) that describes where to find each version's `pg_config` tool.

If a new minor Postgres version is released in the future you can simply run `cargo pgrx init [args]` again, and your local version will be updated, preserving all existing databases and configuration.

```console
$ cargo pgrx init -h
Initialize pgrx development environment for the first time

Usage: cargo pgrx init [OPTIONS]

Options:
      --pg12 <PG12>                            If installed locally, the path to PG12's `pgconfig` tool, or `download`
                                               to have pgrx download/compile/install it [env: PG12_PG_CONFIG=]
      --pg13 <PG13>                            If installed locally, the path to PG13's `pgconfig` tool, or `download`
                                               to have pgrx download/compile/install it [env: PG13_PG_CONFIG=]
  -v, --verbose...                             Enable info logs, -vv for debug, -vvv for trace
      --pg14 <PG14>                            If installed locally, the path to PG14's `pgconfig` tool, or `download`
                                               to have pgrx download/compile/install it [env: PG14_PG_CONFIG=]
      --pg15 <PG15>                            If installed locally, the path to PG15's `pgconfig` tool, or `download`
                                               to have pgrx download/compile/install it [env: PG15_PG_CONFIG=]
      --pg16 <PG16>                            If installed locally, the path to PG16's `pgconfig` tool, or `download`
                                               to have pgrx download/compile/install it [env: PG16_PG_CONFIG=]
      --base-port <BASE_PORT>                  Base port number
      --base-testing-port <BASE_TESTING_PORT>  Base testing port number
      --configure-flag <CONFIGURE_FLAG>        Additional flags to pass to the configure script
      --valgrind                               Compile PostgreSQL with the necessary flags to detect a good amount of
                                               memory errors when run under Valgrind
  -j, --jobs <JOBS>                            Allow N make jobs at once
  -h, --help                                   Print help (see more with '--help')
  -V, --version                                Print version
```

## Creating a new Extension

```rust
$ cargo pgrx new example
$ ls example/
Cargo.toml  example.control  sql  src
```

`cargo pgrx new <extname>` is an easy way to get started creating a new extension. It's similar to `cargo new <name>`, but does the additional things necessary to support building a Rust Postgres extension.

If you'd like to create a "background worker" instead, specify the `--bgworker` argument.

`cargo pgrx new` does not initialize the directory as a git repo, but it does create a `.gitignore` file in case you decide to do so.

> **Workspace users:** `cargo pgrx new $NAME` will create a `$NAME/.cargo/config.toml`, you should move this into your workspace root as `.cargo/config.toml`.
>
> If you don't, you may experience unnecessary rebuilds using tools like Rust-Analyzer, as it will use the wrong `rustflags` option.

```console
$ cargo pgrx new --help
cargo-pgrx-new 0.5.0
PgCentral Foundation, Inc. <contact@pgcentral.org>
Create a new extension crate

USAGE:
    cargo pgrx new [OPTIONS] <NAME>

ARGS:
    <NAME>    The name of the extension

OPTIONS:
    -b, --bgworker    Create a background worker template
    -h, --help        Print help information
    -v, --verbose     Enable info logs, -vv for debug, -vvv for trace
    -V, --version     Print version information
```

## Managing Your Postgres Installations

```console
$ cargo pgrx status all
Postgres v11 is stopped
Postgres v12 is stopped
Postgres v13 is stopped
Postgres v14 is stopped
Postgres v15 is stopped

$ cargo pgrx start all
    Starting Postgres v11 on port 28811
    Starting Postgres v12 on port 28812
    Starting Postgres v13 on port 28813
    Starting Postgres v14 on port 28814
    Starting Postgres v15 on port 28815

$ cargo pgrx status all
Postgres v11 is running
Postgres v12 is running
Postgres v13 is running
Postgres v14 is running
Postgres v15 is running

$ cargo pgrx stop all
    Stopping Postgres v11
    Stopping Postgres v12
    Stopping Postgres v13
    Stopping Postgres v14
    Stopping Postgres v15
```

`cargo pgrx` has three commands for managing each Postgres installation: `start`, `stop`, and `status`. Additionally, `cargo pgrx run` (see below) will automatically start its target Postgres instance if not already running.

When starting a Postgres instance, `pgrx` starts it on port `28800 + PG_MAJOR_VERSION`, so Postgres 11 runs on `28811`, 12 on `28812`, etc. Additionally, the first time any of these are started, it'll automatically initialize a `PGDATA` directory in `~/.pgrx/data-[11 | 12 | 13 | 14 | 15]`. Doing so allows `pgrx` to manage either Postgres versions it installed or ones already on your computer, and to make sure that in the latter case, `pgrx` managed versions don't interfere with what might already be running. The locale of the instance is `C.UTF-8` (or equivalently, a locale of `C` with a `ctype` of `UTF8` on macOS), or `C` if the `C.UTF-8` locale is unavailable.

`pgrx` doesn't tear down these instances. While they're stored in a hidden directory in your home directory, `pgrx` considers these important and permanent database installations.

Once started, you can connect to them using `psql` (if you have it on your $PATH) like so: `psql -p 28812`. However, you probably just want the `cargo pgrx run` command.

## Compiling and Running Your Extension

```console
$ cargo pgrx run pg13
building extension with features ``
"cargo" "build" "--message-format=json-render-diagnostics"
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s

installing extension
     Copying control file to /home/ana/.pgrx/13.5/pgrx-install/share/postgresql/extension/strings.control
     Copying shared library to /home/ana/.pgrx/13.5/pgrx-install/lib/postgresql/strings.so
    Building for SQL generation with features ``
    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
 Discovering SQL entities
  Discovered 6 SQL entities: 0 schemas (0 unique), 6 functions, 0 types, 0 enums, 0 sqls, 0 ords, 0 hashes, 0 aggregates
     Writing SQL entities to /home/ana/.pgrx/13.5/pgrx-install/share/postgresql/extension/strings--0.1.0.sql
    Finished installing strings
    Starting Postgres v13 on port 28813
    Re-using existing database strings
psql (13.5)
Type "help" for help.

strings=# DROP EXTENSION strings;
ERROR:  extension "strings" does not exist
strings=# CREATE EXTENSION strings;
CREATE EXTENSION
strings=# \df strings.*
                                      List of functions
 Schema  |     Name      | Result data type |           Argument data types            | Type
---------+---------------+------------------+------------------------------------------+------
 strings | append        | text             | input text, extra text                   | func
 strings | return_static | text             |                                          | func
 strings | split         | text[]           | input text, pattern text                 | func
 strings | split_set     | SETOF text       | input text, pattern text                 | func
 strings | substring     | text             | input text, start integer, "end" integer | func
 strings | to_lowercase  | text             | input text                               | func
(6 rows)

strings=# select strings.to_lowercase('PGRX');
 to_lowercase
--------------
 pgrx
(1 row)
```

`cargo pgrx run <pg12 | pg13 | pg14 | pg15 | pg16>` is the primary interface into compiling and interactively testing/using your extension during development.

The very first time you execute `cargo pgrx run pgXX`, it needs to compile not only your extension, but pgrx itself, along with all its dependencies. Depending on your computer, this could take a bit of time (`pgrx` is nearly 200k lines of Rust when counting the generated bindings for Postgres). Afterwards, however (as seen in the above screenshot), it's fairly fast.

`cargo pgrx run` compiles your extension, installs it to the specified Postgres installation as described by its `pg_config` tool, starts that Postgres instance using the same process as `cargo pgrx start pgXX`, and drops you into a `psql` shell connected to a database, by default, named after your extension. From there, it's up to you to create your extension and use it.

This is also the stage where `pgrx` automatically generates the SQL schema for your extension via the `sql-generator` binary.

When you exit `psql`, the Postgres instance continues to run in the background.

For Postgres installations which are already on your computer, `cargo pgrx run` will need write permissions to the directories described by `pg_config --pkglibdir` and `pg_config --sharedir`. It's up to you to decide how to make that happen. While a single Postgres installation can be started multiple times on different ports and different data directories, it does not support multiple "extension library directories".

```console
$ cargo pgrx run --help
Compile/install extension to a pgrx-managed Postgres instance and start psql

Usage: cargo pgrx run [OPTIONS] [PG_VERSION] [DBNAME]

Arguments:
  [PG_VERSION]  Do you want to run against pg12, pg13, pg14, pg15, or pg16? [env: PG_VERSION=]
  [DBNAME]      The database to connect to (and create if the first time).  Defaults to a database with the same name as the current extension name

Options:
  -p, --package <PACKAGE>              Package to build (see `cargo help pkgid`)
      --manifest-path <MANIFEST_PATH>  Path to Cargo.toml
  -v, --verbose...                     Enable info logs, -vv for debug, -vvv for trace
  -r, --release                        Compile for release mode (default is debug)
      --profile <PROFILE>              Specific profile to use (conflicts with `--release`)
      --all-features                   Activate all available features
      --no-default-features            Do not activate the `default` feature
  -F, --features <FEATURES>            Space-separated list of features to activate
      --pgcli                          Use an existing `pgcli` on the $PATH [env: PGRX_PGCLI=]
  -h, --help                           Print help
  -V, --version                        Print version
```

## Connect to a Database

```console
$ cargo pgrx connect
    Re-using existing database strings
psql (13.5)
Type "help" for help.

strings=# select strings.to_lowercase('PGRX');
 to_lowercase
--------------
 pgrx
(1 row)

strings=#
```

If you'd simply like to connect to a managed version of Postgres without re-compiling and installing
your extension, use `cargo pgrx connect <pg12 | pg13 | pg14 | pg15 | pg16>`.

This command will use the default database named for your extension, or you can specify another
database name as the final argument.

If the specified database doesn't exist, `cargo pgrx connect` will create it. Similarly, if
the specified version of Postgres isn't running, it'll be automatically started.

```console
$ cargo pgrx connect --help
Connect, via psql, to a Postgres instance

Usage: cargo pgrx connect [OPTIONS] [PG_VERSION] [DBNAME]

Arguments:
  [PG_VERSION]  Do you want to run against pg12, pg13, pg14, pg15, or pg16? [env: PG_VERSION=]
  [DBNAME]      The database to connect to (and create if the first time).  Defaults to a database with the same name as the current extension name [env: DBNAME=]

Options:
  -p, --package <PACKAGE>              Package to determine default `pg_version` with (see `cargo help pkgid`)
      --manifest-path <MANIFEST_PATH>  Path to Cargo.toml
  -v, --verbose...                     Enable info logs, -vv for debug, -vvv for trace
      --pgcli                          Use an existing `pgcli` on the $PATH [env: PGRX_PGCLI=]
  -h, --help                           Print help
  -V, --version                        Print version
```

## Installing Your Extension Locally

```console
$ cargo pgrx install
building extension with features ``
"cargo" "build" "--message-format=json-render-diagnostics"
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s

installing extension
     Copying control file to /usr/share/postgresql/13/extension/strings.control
     Copying shared library to /usr/lib/postgresql/13/lib/strings.so
    Building for SQL generation with features ``
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
 Discovering SQL entities
  Discovered 6 SQL entities: 0 schemas (0 unique), 6 functions, 0 types, 0 enums, 0 sqls, 0 ords, 0 hashes, 0 aggregates
     Writing SQL entities to /usr/share/postgresql/13/extension/strings--0.1.0.sql
    Finished installing strings
```

If for some reason `cargo pgrx run <PG_VERSION>` isn't your style, you can use `cargo pgrx install` to install your extension
to the Postgres installation described by the `pg_config` tool currently on your `$PATH`.

You'll need write permissions to the directories described by `pg_config --pkglibdir` and `pg_config --sharedir`.  If this
is problematic, use `cargo pgrx install --sudo` which compiles the extension as the current user and copies the extension
files to their proper location using `sudo`, prompting you for your password.

By default, `cargo pgrx install` builds your extension in debug mode. Specifying `--release` changes that.

```console
$ cargo pgrx install --help
Install the extension from the current crate to the Postgres specified by whatever `pg_config` is currently on your $PATH

Usage: cargo pgrx install [OPTIONS]

Options:
  -p, --package <PACKAGE>              Package to build (see `cargo help pkgid`)
      --manifest-path <MANIFEST_PATH>  Path to Cargo.toml
  -v, --verbose...                     Enable info logs, -vv for debug, -vvv for trace
  -r, --release                        Compile for release mode (default is debug)
      --profile <PROFILE>              Specific profile to use (conflicts with `--release`)
      --test                           Build in test mode (for `cargo pgrx test`)
  -c, --pg-config <PG_CONFIG>          The `pg_config` path (default is first in $PATH)
  -s, --sudo                           Use `sudo` to install the extension artifacts
      --all-features                   Activate all available features
      --no-default-features            Do not activate the `default` feature
  -F, --features <FEATURES>            Space-separated list of features to activate
  -h, --help                           Print help
  -V, --version                        Print version
```

## Testing Your Extension

```console
$ cargo pgrx test
"cargo" "test" "--features" " pg_test"
    Finished test [unoptimized + debuginfo] target(s) in 0.07s
     Running unittests (target/debug/deps/spi-312296af509607bc)

running 2 tests
building extension with features ` pg_test`
"cargo" "build" "--features" " pg_test" "--message-format=json-render-diagnostics"
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s

installing extension
     Copying control file to /home/ana/.pgrx/13.5/pgrx-install/share/postgresql/extension/spi.control
     Copying shared library to /home/ana/.pgrx/13.5/pgrx-install/lib/postgresql/spi.so
    Building for SQL generation with features ` pg_test`
    Finished test [unoptimized + debuginfo] target(s) in 0.07s
 Discovering SQL entities
  Discovered 11 SQL entities: 1 schemas (1 unique), 8 functions, 0 types, 0 enums, 2 sqls, 0 ords, 0 hashes, 0 aggregates
     Writing SQL entities to /home/ana/.pgrx/13.5/pgrx-install/share/postgresql/extension/spi--0.0.0.sql
    Finished installing spi
test tests::pg_test_spi_query_by_id_direct ... ok
test tests::pg_test_spi_query_by_id_via_spi ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 1.61s

Stopping Postgres
```

`cargo pgrx test [pg12 | pg13 | pg14 | pg15 | pg16]` runs your `#[test]` and `#[pg_test]` annotated functions using cargo's test system.

During the testing process, `pgrx` starts a temporary instance of Postgres with its `PGDATA` directory in `./target/pgrx-test-data-PGVER/`. This Postgres instance is stopped as soon as the test framework has finished. The locale of the temporary instance is `C.UTF-8` (or equivalently, a locale of `C` with a `ctype` of `UTF8` on macOS), or `C` if the `C.UTF-8` locale is unavailable.

The output is standard "cargo test" output along with some Postgres log output. In the case of test failures, the failure report will include any Postgres log messages generated by that particular test.

Rust `#[test]` functions behave normally, while `#[pg_test]` functions are run **inside** the Postgres instance and have full access to all of Postgres internals. All tests are run in parallel, regardless of their type.

Additionally, a `#[pg_test]` function runs in a transaction that is aborted when the test is finished. As such, any changes it might
make to the database are not preserved.

```console
$ cargo pgrx test --help
Run the test suite for this crate

Usage: cargo pgrx test [OPTIONS] [PG_VERSION] [TESTNAME]

Arguments:
  [PG_VERSION]  Do you want to run against pg12, pg13, pg14, pg15, pg16, or all? [env: PG_VERSION=]
  [TESTNAME]    If specified, only run tests containing this string in their names

Options:
  -p, --package <PACKAGE>              Package to build (see `cargo help pkgid`)
      --manifest-path <MANIFEST_PATH>  Path to Cargo.toml
  -v, --verbose...                     Enable info logs, -vv for debug, -vvv for trace
  -r, --release                        compile for release mode (default is debug)
      --profile <PROFILE>              Specific profile to use (conflicts with `--release`)
  -n, --no-schema                      Don't regenerate the schema
      --all-features                   Activate all available features
      --no-default-features            Do not activate the `default` feature
  -F, --features <FEATURES>            Space-separated list of features to activate
  -h, --help                           Print help
  -V, --version                        Print version
```

## Building an Installation Package

```console
$ cargo pgrx package
building extension with features ``
"cargo" "build" "--release" "--message-format=json-render-diagnostics"
    Finished release [optimized] target(s) in 0.07s

installing extension
     Copying control file to target/release/spi-pg13/usr/share/postgresql/13/extension/spi.control
     Copying shared library to target/release/spi-pg13/usr/lib/postgresql/13/lib/spi.so
    Building for SQL generation with features ``
    Finished release [optimized] target(s) in 0.07s
 Discovering SQL entities
  Discovered 8 SQL entities: 0 schemas (0 unique), 6 functions, 0 types, 0 enums, 2 sqls, 0 ords, 0 hashes, 0 aggregates
     Writing SQL entities to target/release/spi-pg13/usr/share/postgresql/13/extension/spi--0.0.0.sql
    Finished installing spi
```

`cargo pgrx package [--debug]` builds your extension, in `--release` mode, to a directory structure in
`./target/[debug | release]/extension_name-PGVER` using the Postgres installation path information from the `pg_config`
tool on your `$PATH`.

The intent is that you'd then change into that directory and build a tarball or a .deb or .rpm package.

The directory structure `cargo pgrx package` creates starts at the root of the filesystem, as a package-manager installed
version of Postgres is likely to split `pg_config --pkglibdir` and `pg_config --sharedir` into different base paths.

(In the example screenshot above, `cargo pgrx package` was used to build a directory structure using my manually installed
version of Postgres 12.)

This command could be useful from Dockerfiles, for example, to automate building installation packages for various Linux
distobutions or MacOS Postgres installations.

```console
$ cargo pgrx package --help
Create an installation package directory

Usage: cargo pgrx package [OPTIONS]

Options:
  -p, --package <PACKAGE>              Package to build (see `cargo help pkgid`)
      --manifest-path <MANIFEST_PATH>  Path to Cargo.toml
  -v, --verbose...                     Enable info logs, -vv for debug, -vvv for trace
  -d, --debug                          Compile for debug mode (default is release)
      --profile <PROFILE>              Specific profile to use (conflicts with `--debug`)
      --test                           Build in test mode (for `cargo pgrx test`)
  -c, --pg-config <PG_CONFIG>          The `pg_config` path (default is first in $PATH)
      --out-dir <OUT_DIR>              The directory to output the package (default is `./target/[debug|release]/extname-pgXX/`)
      --all-features                   Activate all available features
      --no-default-features            Do not activate the `default` feature
  -F, --features <FEATURES>            Space-separated list of features to activate
  -h, --help                           Print help
  -V, --version                        Print version
```

## Inspect your Extension Schema

If you just want to look at the full extension schema that pgrx will generate, use
`cargo pgrx schema`.

```console
$ cargo pgrx schema --help
Generate extension schema files

Usage: cargo pgrx schema [OPTIONS] [PG_VERSION]

Arguments:
  [PG_VERSION]  Do you want to run against pg12, pg13, pg14, pg15, or pg16?

Options:
  -p, --package <PACKAGE>              Package to build (see `cargo help pkgid`)
      --manifest-path <MANIFEST_PATH>  Path to Cargo.toml
  -v, --verbose...                     Enable info logs, -vv for debug, -vvv for trace
      --test                           Build in test mode (for `cargo pgrx test`)
  -r, --release                        Compile for release mode (default is debug)
      --profile <PROFILE>              Specific profile to use (conflicts with `--release`)
  -c, --pg-config <PG_CONFIG>          The `pg_config` path (default is first in $PATH)
      --all-features                   Activate all available features
      --no-default-features            Do not activate the `default` feature
  -F, --features <FEATURES>            Space-separated list of features to activate
  -o, --out <OUT>                      A path to output a produced SQL file (default is `stdout`)
  -d, --dot <DOT>                      A path to output a produced GraphViz DOT file
      --skip-build                     Skip building a fresh extension shared object
  -h, --help                           Print help
  -V, --version                        Print version
```

## Information about pgx-managed development environment

```console
$ cargo pgrx info --help
Provides information about pgrx-managed development environment

Usage: cargo pgrx info [OPTIONS] <COMMAND>

Commands:
  path       Print path to a base version of Postgres build
  pg-config  Print path to pg_config for a base version of Postgres
  version    Print specific version for a base Postgres version
  help       Print this message or the help of the given subcommand(s)

Options:
  -v, --verbose...  Enable info logs, -vv for debug, -vvv for trace
  -h, --help        Print help
  -V, --version     Print version
```

`cargo pgx info` helps retrieving information about pgx-managed development
environment (such as managed Postgres installations)

## EXPERIMENTAL: Versioned shared-object support

`pgrx` experimentally supports the option to produce a versioned shared library. This allows multiple versions of the
extension to be installed side-by-side, and can enable the deprecation (and removal) of functions between extension
versions. There are some caveats which must be observed when using this functionality. For this reason it is currently
experimental.

### Activation

Versioned shared-object support is enabled by removing the `module_pathname` configuration value in the extension's
`.control` file.

### Concepts

Postgres has the implicit requirement that C extensions maintain ABI compatibility between versions. The idea behind
this feature is to allow interoperability between two versions of an extension when the new version is not ABI
compatible with the old version.

The mechanism of operation is to version the name of the shared library file, and to hard-code function definitions to
point to the versioned shared library file. Without versioned shared-object support, the SQL definition of a C function
would look as follows:

```SQL
CREATE OR REPLACE FUNCTION "hello_extension"() RETURNS text /* &str */
STRICT
LANGUAGE c /* Rust */
AS 'MODULE_PATHNAME', 'hello_extension_wrapper';
```

`MODULE_PATHNAME` is replaced by Postgres with the configured value in the `.control` file. For pgrx-based extensions,
this is  usually set to `$libdir/<extension-name>`.

When using versioned shared-object support, the same SQL would look as follows:

```SQL
CREATE OR REPLACE FUNCTION "hello_extension"() RETURNS text /* &str */
STRICT
LANGUAGE c /* Rust */
AS '$libdir/extension-0.0.0', 'hello_extension_wrapper';
```

Note that the versioned shared library is hard-coded in the function definition. This corresponds to the
`extension-0.0.0.so` file which `pgrx` generates.

It is important to note that the emitted SQL is version-dependent. This means that all previously-defined C functions
must be redefined to point to the current versioned-so in the version upgrade script. As an example, when updating the
extension version to 0.1.0, the shared object will be named `<extension-name>-0.1.0.so`, and `cargo pgrx schema` will
produce the following SQL for the above function:

```SQL
CREATE OR REPLACE FUNCTION "hello_extension"() RETURNS text /* &str */
STRICT
LANGUAGE c /* Rust */
AS '$libdir/extension-0.1.0', 'hello_extension_wrapper';
```

This SQL must be used in the upgrade script from `0.0.0` to `0.1.0` in order to point the `hello_extension` function to
the new shared object. `pgrx` _does not_ do any magic to determine in which version a function was introduced or modified
and only place it in the corresponding versioned so file. By extension, you can always expect that the shared library
will contain _all_ functions which are still defined in the extension's source code.

This feature is not designed to assist in the backwards compatibility of data types.

### `@MODULE_PATHNAME@` Templating

In case you are already providing custom SQL definitions for Rust functions, you can use the `@MODULE_PATHNAME@`
template in your custom SQL. This value will be replaced with the path to the actual shared object.

The following example illustrates how this works:

```rust
#[pg_extern(sql = r#"
    CREATE OR REPLACE FUNCTION tests."overridden_sql_with_fn_name"() RETURNS void
    STRICT
    LANGUAGE c /* Rust */
    AS '@MODULE_PATHNAME@', '@FUNCTION_NAME@';
"#)]
fn overridden_sql_with_fn_name() -> bool {
    true
}
```

### Caveats

There are some scenarios which are entirely incompatible with this feature, because they rely on some global state in
Postgres, so loading two versions of the shared library will cause trouble.

These scenarios are:
- when using shared memory
- when using query planner hooks

## Compiler Version Dependence

The version of the Rust compiler and toolchain used to build `cargo-pgrx` must be
the same as the version used to build your extension.

Several subcommands (including `cargo pgrx schema`, `cargo pgrx test`, `cargo pgrx
install`, ...) will produce an error message if these do not match.

Although this may be relaxed in the future, currently schema generation involves
`dlopen`ing the extension and calling `extern "Rust"` functions on
`#[repr(Rust)]` types. Generally, the appropriate way to fix this is reinstall
`cargo-pgrx`, using a command like the following

```console
$ cargo install --force --locked cargo-pgrx
```

Possibly with a explicit `--version`, if needed.

If you are certain that in this case, it is fine, you may set
`PGRX_IGNORE_RUST_VERSIONS` in the environment (to any value other than `"0"`),
and the check will be bypassed. However, note that while the check is not
fool-proof, it tries to be fairly liberal in what it allows.

See <https://github.com/pgcentralfoundation/pgrx/issues/774> and <https://github.com/pgcentralfoundation/pgrx/pull/873>
for further information.
