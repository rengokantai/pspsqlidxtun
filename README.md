#### pspsqlidxtun
#####EXPLAIN Keyword
######8Various Options of EXPLAIN
```
ANAYLZE->f
VERBOSE->t
COSTS->t
BUFFERS->f
TIMING->t
FORMAT->text   (text/json/xml/yaml)
```
######10 Environment Setup
for vps
```
vim /etc/postgresql/9.5/main/postgresql.conf
```
edit
```
listen_addresses = '*'
```
then
```
vim /etc/postgresql/9.5/main/pg_hba.conf
```
edit
```
host    all             all          0.0.0.0/0               md5
host    all             all             ::/0                 md5
```
enable iptables
```
iptables -A INPUT -p tcp -m tcp --dport 5432 -j ACCEPT
iptables -A OUTPUT -p tcp -m tcp --dport 5432 -j ACCEPT
```
then import data
```
wget http://bit.ly/pagilia-dl -O test.zip
unzip test.zip
pg_restore -d dvdrental dvdrental.tar
```
######Query Plan and Cost
cost= diskpages read* seq_page_cost + rows scanned* cpu_tuple_cost

```
select relpages,reltuples from pg_class where relname='film';  -- 54,1000
```
seq_page_cost = estimate of the cost of a disk page fetch,default=1
cpu_tuple_cost = estimate of the cost of processing each row,default=0.01
hence total = 54*0.1 +10=64

######Query Plan and Conditions
where clause: id < 10: index cond(index scan)
id> 10: filter(seq scan)

#####Query Performance with Indexes
######Types of Indexes
btree  
hash  
generalized inverted index(gin)
generalized search tree index(gist)
######Btree
```
< > = <= BETWEEN IN
```
######B-Tree Index - Disadvantages
inserting slower data (insert,delete,update)
######Multicolumn Index and Cover Index
maximum 32 columns
######Index and Improved Performance
basic
```
select title,length,rating,replacement_cost,rental_rate from film where length between 60 and 70 and rating = 'G';
```
create index
```
create index idx_film_length ON film(length);
```
######Multicolumn Index
```
create index idx_film_length_rating ON film(length,rating);
```
analyze it:
```
explain analyze select title,length,rating,replacement_cost,rental_rate from film where length between 60 and 70 and rating = 'G';
```
check Heap Blocks = x (the small the better)
######Order of Column in Index
first: heap scan second:filter
######Index Maintenance with REINDEX
```
REINDEX INDEX idx_film_cover; / DROP INDEX idx_film_length;
REINDEX TABLE film;
```
#####Complex Queries
######Difference Between Key, Index, and Constraint
postgres automatically creates a unique index when a unique constraint or primary key is defined on table
######Demo: Unique Index
create a new table
```
create table SampleTable(
id integer primary key,
firstcol character varying(30),
secondcol integer
);
```
click constraints,you can see `sampletable_pkey`
```
ALTER TABLE public.sampletable ADD CONSTRAINT sampletable_pkey PRIMARY KEY(id);
```
we cannot see any index.
######Primary Key Constraint and Catalog Tables
show all indexes(including invisible constraint index)
```
select idx.indrelid :: REGCLASS as table_name,
i.relname as index.name,
idx.indisunique as is_unique,
idx.indisprimary as is_primary
from pg_index as idx join pg_class as i on i.old = idx.indexrelid
where idx.indrelid = 'sampletable'::reglcass;
```
create index
```
create index idx_sampletable_id on SampleTable(id);
```
######Unique Constraint and Catalog Tables
```
alter table sampletable add constraint sampletable_firstcol unique(firstcol);
```
exclipitly
```
create unique index unq_sampletable_firstcol on sampletable(firstcol);
```
######Case Insensitive Search
change table accept insensive search
```
select * from film where lower(title) = lower('arizona');
```
but perf is not good
###### Case Insensitive Search and Performance
```
create index film_title_search_lower on film (lower(title));
```
###### Partial Index
````
create index film_first on film(length) where length<60;
```
#####Large Database
######Disable Autocommit
create table
```
create table ke(
film_id integer not null,
title character varying(255) not null,
description text,
release_year year,
language_id smallint not null,
rental_rate numeric(4,2) default 1.11 not null,
last_update timestamp without time zone default now() not null,
fulltext tsvector not null
);
```
######Copy Command
```
copy ke to stdout;
copy ke to stdout(delimiter ',');
```
save csv
```
copy ke to 'd:/ke.csv' csv
copy ke to 'd:/ke.csv' csv header  #with header
copy ke from 'd:/ke.csv' delimiters ','
```
######Summary
