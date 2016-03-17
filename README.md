#### orhoazredshift
#####1
######data warehouse part1
OLTP vs OLAP  
OLTP:  
RDBMS  
Fast, ad-hoc queries, haavliy concurrent, heavily normalized via 3NF, row-oriented  
OLAP:  
DW 
column-oriented  
hugely-scalable  


#####2
###### conf SQL workbench

go to connect client, download JDBC 4.1 driver.  
download SQL workbench, create a new connection profile, name redshift, choose jdbd4.1.jar.  
Connect,choose auto commit, copy jdbc link. To prevent IDLE,Add connect script.  
Statement to keep connection alive:  
```
select version();
```
######Loading into S3
```
aws s3 cb . s3://yd-redshift/data --recursive
```
```
select distinct(tablename) from pg_table_def where schemaname = 'public';
```
######loading into redshift cluster p2
```
set wlm_query_slot_count to 2;
```

copy csv data into redshift:
```
copy part from 's3://yd-redshift/data/part-csv.tbl' 
credentials 'aws_access_key_id=AK;aws_secret_access_key=HY'
csv
null as '\000';
```

setting a manifest file and upload to s3:
```
{
  "entries": [
    {"url":"s3://yd-redshift/data/customer-fw.tbl-000"},
    {"url":"s3://yd-redshift/data/customer-fw.tbl-001"},
    {"url":"s3://yd-redshift/data/customer-fw.tbl-002"},
    {"url":"s3://yd-redshift/data/customer-fw.tbl-003"},
    {"url":"s3://yd-redshift/data/customer-fw.tbl-004"},    
    {"url":"s3://yd-redshift/data/customer-fw.tbl-005"},
    {"url":"s3://yd-redshift/data/customer-fw.tbl-006"}, 
    {"url":"s3://yd-redshift/data/customer-fw.tbl-007"}
 ]
}
```
copy to s3:
```
aws s3 cp customer-fw-manifest s3://yd-redshift/data/
```

SQL to retrive data from manifest file:
```
copy customer
from 's3://yd-redshift/data/customer-fw-manifest'
region 'us-east-1'
credentials 'aws_access_key_id=AKIAIXVX2XIKPDGJJSMQ;aws_secret_access_key=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'
fixedwidth 'c_custkey:10, c_name:25, c_address:25, c_city:10, c_nation:15, c_region :12, c_phone:15,c_mktsegment:10'
maxerror 10
acceptinvchars as '^'
manifest;
```

######loading into redshift cluster p3
```
select query,step,rows,workmem,label,is_diskbased
from svl_query_summary
where query=100
order by workmem desc;
```
