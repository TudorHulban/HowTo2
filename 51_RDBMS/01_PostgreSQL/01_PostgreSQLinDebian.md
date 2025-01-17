# PostgreSQL: Install and first steps with Debian

## 1. Check if PostgreSQL is already installed

```sh
sudo which psql
# or
service postgresql status
```

## 2. Add line for repo

```sh
deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main
sudo touch /etc/apt/sources.list.d/pgdg.list
sudo vi /etc/apt/sources.list.d/pgdg.list
```

For Debian 9.x:

```sh
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get install postgresql-11
```

## 3. Install binaries

```sh
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install postgresql-9.6 
(pgadmin3)
```

Command would create user postgres and initial cluster using command:

```sh
/usr/lib/postgresql/9.6/bin/initdb -D /var/lib/postgresql/9.6/main --auth-local peer --auth-host md5
```

Configuration file in:

```sh
/etc/postgresql/9.6/main/postgresql.conf 
# log file in:
/var/log/postgresql/postgresql-9.6-main.log
```

Location of files:

```sql
SELECT name, setting FROM pg_settings WHERE category = 'File Locations';
```

## 4. Stop server

```sh
sudo su - postgres
/usr/lib/postgresql/9.6/bin/pg_ctl stop -D /var/lib/postgresql/9.6/main/
```

## 5. Start server

```sh
sudo service postgresql-17 restart
```

## 6. Enable TCP/IP connections

```sh
vi /var/lib/pgsql/17/data/postgresql.conf 
# modify: 
listen_addresses = '*'
```

### Alternate ports

5433 - next logical choice, which is often unassigned by default.  
5434-5440 - still in a range that’s likely open and unused.  
5470-5499 - ports in this range are typically available on Linux systems.

Restart service so new configuration taakes effect:

```sh
sudo systemctl restart postgresql-17
```

## 7. Add the IP to connect from

```sh
vi /var/lib/pgsql/17/data/pg_hba.conf
# add:
host    all             all             192.168.1.0/24          scram-sha-256
```

Restart service:

```sh
sudo service postgresql restart
```

## 6. Change password of default user

```sh
sudo su postgres
psql
```

```sql
alter user postgres with password 'admin';
\q
```

## 7. To remove cluster

```sh
stop database server service
delete cluster folder
```

## 8. Operations

List tablespaces:

```sql
\db[+]
```

List databases:

```sql
\l
/* or */
SELECT datname FROM pg_database WHERE datistemplate = false;
```

Create user:

```sql
create user gogs with password 'gogs';
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO gogs;
create: CREATE TABLESPACE tbl_gogs LOCATION '/rich1/pdata';
create database db_gogs owner gogs tablespace tbl_gogs; 

connect \c db_gogs, tables \dt
show data:
show data_directory
SELECT pg_relation_filepath('tablename');
/* see also: https://www.postgresql.org/docs/9.0/static/storage-file-layout.html */
select * from information_schema.tables where table_schema = 'public'
```

## 9. Remove PostgreSQL

```sh
sudo apt-get --purge remove postgresql
```

## 1o. Docker image

```sh
sudo docker pull postgres:10.1
```

Check image id:

```sh
sudo docker images
```

Create container and map ports to host:

```sh
sudo docker run -d -p 5432:5432 --name {container name} {image ID, 4 digits}
```

## 11. For client development

```sh
# consider sudo apt-get remove libpq5
sudo apt-get install libpq-dev
```

## 12. Populate table with mock data

Create function to create table and insert random values:

```sql
create or replace function mocktable(ATableName char,ARows int) returns int 
as $$
declare 
i integer := 0;
randstring char(16) := '';

begin
execute 'create table ' || ATableName ||'(ID INT PRIMARY KEY NOT NULL,DESCRIPTION TEXT NOT NULL);';

if ARows > 0 then 
loop 
exit when i = ARows;
i:= i + 1;

randstring:= array_to_string(array(select substr('ABCDEFGHIJKLMNPQRSTUVWXYZ123456789',((random()*(36-1)+1)::integer),1) from generate_series(1,16)),'');

execute 'insert into ' || ATableName || '(id, description) values ('|| i ||','''|| randstring ||''')';
end loop;
end if;

return i;
end; $$
language plpgsql
```

Use it as:

```sql
select mocktable('posts', 10000);
```  

## 13. Verify connections

```sql
SELECT * FROM pg_stat_activity;
```

## 14. Cloud

```html
https://access.crunchydata.com/documentation/crunchy-postgres-containers/2.4.0/
```

## Resources

```html
https://opensource.com/article/17/6/ansible-postgresql-operations
```
