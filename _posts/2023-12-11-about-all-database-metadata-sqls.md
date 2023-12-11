---
layout: post
title: "各个类型数据库的元数据查询sql分享"
categories: tech-share
author: "papadave"
---
近期的工作都是围绕元数据相关，以下是我整理的一些各个数据库sql语句:  
MySQL表信息:  
``` 
SELECT table_name as TABLE_NAME,TABLE_COMMENT as TABLE_DESC from information_schema.tables where TABLE_TYPE = 'BASE TABLE' AND table_schema=? 
```  
MySQL列信息:  
``` 
select col.table_schema as DB_NAME,
		col.table_name as TABLE_NAME, column_name as COLUMN_NAME, 
		column_comment as COLUMN_COMMENT,column_type as COLUMN_TYPE, 
		table_comment as TABLE_DESC, ordinal_position as ORDINAL_POSITION,
		is_nullable as IS_NULLABLE , CHARACTER_MAXIMUM_LENGTH as char_length,
		NUMERIC_PRECISION as DATA_PRECISION, NUMERIC_SCALE as DATA_SCALE,
		COLUMN_DEFAULT as DATA_DEFAULT,
		CASE 
			WHEN column_key = 'PRI' THEN 1 ELSE 0 
		END AS IF_PRIMARY_KEY,
		CASE 
			WHEN column_key = 'UNI' THEN 1 ELSE 0 
		END AS IF_UNIQUE,
		CASE 
			WHEN column_key = 'MUL' THEN 1 ELSE 0 
		END AS IF_PRIMARY_KEY
		from information_schema.columns col 
		INNER JOIN information_schema.TABLES tab on tab.table_schema = col.table_schema and 
		tab.table_name=col.table_name 
where col.table_schema=? 
```  
达梦表信息:  
```
select TABLE_NAME,COMMENTS as TABLE_DESC from ALL_TAB_COMMENTS where TABLE_TYPE = 'TABLE' and OWNER =?
```  
达梦列信息:  
```
 select A.OWNER as DB_NAME,A.TABLE_NAME as TABLE_NAME, 
        A.COLUMN_NAME as COLUMN_NAME,B.comments as COLUMN_COMMENT, 
        A.DATA_TYPE as COLUMN_TYPE,C.COMMENTS as TABLE_DESC, 
        A.COLUMN_ID as ORDINAL_POSITION,A.NULLABLE as IS_NULLABLE, 
        A.CHAR_LENGTH as CHAR_LENGTH,A.DATA_SCALE as DATA_SCALE, 
        A.DATA_PRECISION as DATA_PRECISION, 
        A.DATA_DEFAULT AS DATA_DEFAULT,
        CASE
	        WHEN ap.CONSTRAINT_TYPE = 'P' THEN 1
	        ELSE 0
    	END AS IF_PRIMARY_KEY,
		CASE
	        WHEN au.CONSTRAINT_TYPE = 'U' THEN 1
	        ELSE 0
	    END AS IF_UNIQUE
        from all_tab_columns A 
        left join  all_col_comments B ON A.TABLE_NAME=B.TABLE_NAME AND A.COLUMN_NAME=B.COLUMN_NAME AND A.OWNER =B.SCHEMA_NAME 
        left join ALL_TAB_COMMENTS C ON A.TABLE_NAME=C.TABLE_NAME AND C.OWNER=A.OWNER 
        left join (SELECT DISTINCT ACC.INDEX_OWNER, ACC.INDEX_NAME, ACC.TABLE_NAME, ACC.COLUMN_NAME,AC.CONSTRAINT_TYPE FROM all_ind_columns ACC LEFT JOIN ALL_CONSTRAINTS AC on ACC.TABLE_OWNER = AC.OWNER AND ACC.TABLE_NAME = AC.TABLE_NAME and acc.index_name = ac.index_name WHERE AC.CONSTRAINT_TYPE = 'P') ap
        on ap.INDEX_OWNER = a.owner and ap.TABLE_NAME = a.table_name and ap.COLUMN_NAME = a.COLUMN_NAME
        left join (SELECT DISTINCT ACC.INDEX_OWNER, ACC.INDEX_NAME, ACC.TABLE_NAME, ACC.COLUMN_NAME,AC.CONSTRAINT_TYPE FROM all_ind_columns ACC LEFT JOIN ALL_CONSTRAINTS AC on ACC.TABLE_OWNER = AC.OWNER AND ACC.TABLE_NAME = AC.TABLE_NAME and acc.index_name = ac.index_name WHERE AC.CONSTRAINT_TYPE = 'U') au
        on au.INDEX_OWNER = a.owner and au.TABLE_NAME = a.table_name and au.COLUMN_NAME = a.COLUMN_NAME
        where a.owner = ? 
```  
Oracle表信息:  
``` 
SELECT t.TABLE_NAME, c.COMMENTS AS TABLE_DESC FROM ALL_TABLES t LEFT join all_tab_comments c ON t.TABLE_NAME  = c.TABLE_NAME AND t.owner = c.OWNER  WHERE t.OWNER = ? 
```  
Oracle列信息:  
```
SELECT col.OWNER AS DB_NAME, col.TABLE_NAME, tcom.COMMENTS AS TABLE_DESC,
		col.COLUMN_NAME , COM.comments AS COLUMN_COMMENT, col.DATA_TYPE as COLUMN_TYPE,
		col.NULLABLE AS IS_NULLABLE, col.COLUMN_ID AS ORDINAL_POSITION,
		col.CHAR_LENGTH, col.DATA_PRECISION, col.DATA_DEFAULT,  col.DATA_SCALE, 
		CASE
	        WHEN ap.CONSTRAINT_TYPE = 'P' THEN 1
	        ELSE 0
    	END AS IF_PRIMARY_KEY,
		CASE
	        WHEN au.CONSTRAINT_TYPE = 'U' THEN 1
	        ELSE 0
	    END AS IF_UNIQUE
		FROM ALL_TAB_COLUMNS col 
		LEFT JOIN ALL_COL_COMMENTS com 
		ON col.TABLE_NAME = com.TABLE_NAME AND col.COLUMN_NAME  = com.COLUMN_NAME 
		AND col.owner = com.owner
		LEFT JOIN ALL_TAB_COMMENTS tcom 
		ON col.TABLE_NAME = tcom.TABLE_NAME AND col.OWNER = tcom.OWNER
		LEFT JOIN (SELECT DISTINCT ACC.OWNER ,ACC.TABLE_NAME, ACC.COLUMN_NAME,AC.CONSTRAINT_TYPE FROM ALL_CONS_COLUMNS ACC LEFT JOIN ALL_CONSTRAINTS AC ON ACC.OWNER = AC.OWNER AND ACC.CONSTRAINT_NAME = AC.CONSTRAINT_NAME WHERE AC.CONSTRAINT_TYPE = 'P') ap
		ON col.OWNER = ap.owner AND col.table_name = ap.table_name AND col.COLUMN_NAME = ap.column_name
		LEFT JOIN (SELECT DISTINCT ACC.OWNER ,ACC.TABLE_NAME, ACC.COLUMN_NAME,AC.CONSTRAINT_TYPE FROM ALL_CONS_COLUMNS ACC LEFT JOIN ALL_CONSTRAINTS AC ON ACC.OWNER = AC.OWNER AND ACC.CONSTRAINT_NAME = AC.CONSTRAINT_NAME WHERE AC.CONSTRAINT_TYPE = 'U') au
		ON col.OWNER = au.owner AND col.table_name = au.table_name AND col.COLUMN_NAME = au.column_name
	WHERE col.owner = ?
```  
DB2表信息:  
```
SELECT TABNAME AS TABLE_NAME, REMARKS AS TABLE_DESC FROM syscat.tables WHERE TYPE = 'T' AND TABSCHEMA = ?
```
DB2列信息:  
```
SELECT sc.COLNO AS ORDINAL_POSITION, sc.tabschema AS DB_NAME,
		sc.tabname AS TABLE_NAME, st.remarks AS TABLE_DESC, 
		sc.colname AS COLUMN_NAME, sc.typename AS column_type,
		sc.LENGTH AS CHAR_LENGTH, sc.NULLS AS IS_NULLABLE,
		sc.DEFAULT AS DATA_DEFAULT, 
		sc.scale as DATA_SCALE,
		SYSIBMCOL.NUMERIC_PRECISION AS DATA_PRECISION,
		sc.REMARKS AS column_comment,
		CASE
	        WHEN sysip.UNIQUERULE = 'P' THEN 1
	        ELSE 0
    	END AS IF_PRIMARY_KEY,
    	CASE
	        WHEN sysiu.UNIQUERULE = 'U' THEN 1
	        ELSE 0
	    END AS IF_UNIQUE,
	    CASE
	        WHEN sysir.UNIQUERULE = 'R' THEN 1
	        ELSE 0
	    END AS IF_INDEX
		FROM SYSCAT.COLUMNS sc
		LEFT JOIN syscat.tables st
		on sc.tabname = st.tabname AND sc.TABSCHEMA = st.TABSCHEMA 
		LEFT JOIN sysibm.columns SYSIBMCOL
		ON sc.tabname = SYSIBMCOL.table_name AND sc.COLNAME = SYSIBMCOL.column_name 
		AND sc.TABSCHEMA = SYSIBMCOL.table_schema 
		LEFT JOIN (SELECT * FROM SYSIBM.sysindexes WHERE uniquerule = 'P') sysip
		ON sc.TABSCHEMA = sysip.creator AND sc.tabname = sysip.tbname 
		AND sysip.colnames LIKE '%'||sc.colname||'%'  
		LEFT JOIN (SELECT * FROM SYSIBM.sysindexes WHERE uniquerule = 'U') sysiu
		ON sc.TABSCHEMA = sysiu.creator AND sc.tabname = sysiu.tbname 
		AND sysiu.colnames LIKE '%'||sc.colname||'%'  
		LEFT JOIN (SELECT * FROM SYSIBM.sysindexes WHERE uniquerule = 'R') sysir
		ON sc.TABSCHEMA = sysir.creator AND sc.tabname = sysir.tbname 
		AND sysir.colnames LIKE '%'||sc.colname||'%' 
WHERE sc.TABSCHEMA = ?
```
PosgreSQL表信息:  
```
select table_name as "TABLE_NAME",
    obj_description((table_schema||'.'||quote_ident(table_name))::regclass) as "TABLE_DESC"
from information_schema.tables where table_type = 'BASE TABLE' and table_schema = ?
```
PosgreSQL列信息:  
```
select table_schema AS "DB_NAME",
	table_name as "TABLE_NAME", column_name as "COLUMN_NAME", 
	ordinal_position as "ORDINAL_POSITION", column_default AS "DATA_DEFAULT",
	character_maximum_length as "CHAR_LENGTH",
	numeric_precision as "DATA_PRECISION", NUMERIC_SCALE as "DATA_SCALE",
	is_nullable as "IS_NULLABLE", data_type AS "COLUMN_TYPE",
	obj_description((table_schema||'.'||quote_ident(table_name))::regclass) as "TABLE_DESC",
	col_description((table_schema||'.'||quote_ident(table_name))::regclass, ordinal_position::int) as "COLUMN_COMMENT"
	FROM information_schema.columns where table_schema = ?
```  
MS SQL Server 表信息:
```
SELECT table_name as TABLE_NAME,table_type AS TABLE_TYPE from INFORMATION_SCHEMA.tables WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_SCHEMA = ? 
```  
MS SQL Server 列信息:  
```
SELECT ORDINAL_POSITION, TABLE_CATALOG as DB_NAME, 
		table_name as TABLE_NAME, DATA_TYPE as COLUMN_TYPE, 
		COLUMN_NAME, IS_NULLABLE, 
		CHARACTER_MAXIMUM_LENGTH as CHAR_LENGTH,
		NUMERIC_PRECISION as DATA_PRECISION,
		NUMERIC_SCALE as DATA_SCALE,
		COLUMN_DEFAULT as DATA_DEFAULT
		FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_SCHEMA = ?
```  
ClickHouse 表信息:  
```
select name as TABLE_NAME,comment as TABLE_DESC from system.tables where database = ?
```  
ClickHouse 列信息:  
```
select col.table_schema as DB_NAME, 
	col.table_name as TABLE_NAME, 
	col.column_name as COLUMN_NAME, 
	col.column_comment as COLUMN_COMMENT, 
	col.column_type as COLUMN_TYPE, 
	tab.comment as TABLE_DESC,ordinal_position as ORDINAL_POSITION, 
	case 
		when is_nullable=1 then 'Y'
		when is_nullable=0 then 'N'
	end as IS_NULLABLE, 
	col.column_default as DATA_DEFAULT,
	scol.is_in_primary_key as IF_PRIMARY_KEY
	from INFORMATION_SCHEMA.COLUMNS col 
	LEFT JOIN system.tables tab on tab.database=col.table_schema and tab.name=col.table_name
	LEFT JOIN system.columns scol on scol.database=col.table_schema and scol.table=col.table_name and scol.name = col.column_name
where col.table_schema=?
```
