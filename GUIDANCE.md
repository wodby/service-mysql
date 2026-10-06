# MySQL on Wodby

What Wodby and the image set up for this database service. Check it before creating databases or users by hand, or adding connection settings to an application.

## Database, user and passwords

Wodby manages the database and its user; the application does not create them.

- One database and one user are created for the environment, both named after the application and the environment. The user is granted all privileges on that database only.
- The database uses `utf8mb4` with the collation `utf8mb4_0900_ai_ci`.
- The user's password and the `root` password are generated once per environment (tokens `password` and `root_password`). The container has the root password in `MYSQL_ROOT_PASSWORD`; `root` may connect from any host (`MYSQL_ROOT_HOST`).
- Further databases and users are added on Wodby, which runs the same create and grant actions. A user created by hand with a name Wodby later needs, but another password, makes the create action fail rather than replace it.

## How a linked service reaches it

- Host: the name of this app service inside the environment. Port: `3306`.
- A service linked to this one receives the host, port, database name, user name and password as environment variables defined by its own link (for example `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD` on the PHP service). Read those in the application; do not hardcode them and do not use `root` from the application.

## Server configuration

The image is the official MySQL image with a Wodby entrypoint. On every start it writes `/etc/mysql/conf.d/zz-wodby.cnf` from the template `/etc/gotpl/my.cnf.tmpl` and environment variables. Two ways to change settings, neither of them editing the generated file:

- Set a variable on this service: `MYSQL_INNODB_BUFFER_POOL_SIZE`, `MYSQL_MAX_CONNECTIONS`, `MYSQL_MAX_ALLOWED_PACKET`, `MYSQL_WAIT_TIMEOUT`, `MYSQL_TRANSACTION_ISOLATION`, `MYSQL_CHARACTER_SET_SERVER`, `MYSQL_COLLATION_SERVER`. These are written only when set: `MYSQL_SQL_MODE`, `MYSQL_SLOW_QUERY_LOG`, `MYSQL_LONG_QUERY_TIME`, `MYSQL_PERFORMANCE_SCHEMA`, `MYSQL_INNODB_REDO_LOG_CAPACITY`, `MYSQL_INNODB_IO_CAPACITY`, `MYSQL_LOWER_CASE_TABLE_NAMES`.
- Override the service's config file `config` ("MySQL config"), which replaces the template itself, for a setting no variable covers.

A change applies with the next deployment of the service. The service runs a single instance.

## Data, backups and imports

- Data is on the `data` volume, mounted at `/var/lib/mysql`.
- The backup is a gzipped SQL dump of the environment's database. Tables listed in its "excluded table contents" option (names or `LIKE` patterns) keep their definition and lose their rows.
- The database import replaces the data volume: the dump is loaded while a new, empty data directory is initialized. It accepts a `.sql` or `.mysql` file, or a `.gz`, `.tar.gz`, `.tgz` or `.zip` holding exactly one such file.

## Check the result

In the database container:

- `mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e 'SHOW DATABASES;'` lists the databases.
- `mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e "SHOW VARIABLES LIKE 'max_connections';"` shows a setting as the server runs it.
