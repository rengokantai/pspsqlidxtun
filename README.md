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
```
hence total = 54*0.1 +10=64

