Oracle Database
===
[原文網址](https://dba.stackexchange.com/questions/61480/how-to-mirror-the-privileges-of-an-user-to-another-in-oracle-database)
###### tags: `DBA` `privilege` `DBMS` `METADATA` `GET_GRANTED_DDL` `CREATE` `VIEW`
## 2019/09/03
<p>
開發關務系統時，發現.net MVC entity frame work 不支援 oracle synonyms
，那就乾脆建一個新的帳號，然後create view就好，結果又跟我說權限不足create view，但是這個帳號本身已經是dba身份了！所以乾脆就想說copy一個dba帳號的權限
</p>
<p>
然後就有熱心的開發者說下面這段sql可以查詢到某帳號的所有權限及grant 語法，接著用文字編輯器取代該帳號就行了！
</p>

```sql
SELECT DBMS_METADATA.GET_GRANTED_DDL('ROLE_GRANT','AAA') 
FROM DUAL;

SELECT DBMS_METADATA.GET_GRANTED_DDL('SYSTEM_GRANT','AAA') 
FROM DUAL;

SELECT DBMS_METADATA.GET_GRANTED_DDL('OBJECT_GRANT','AAA') 
FROM DUAL;
```
<p>
結果發現前面四行就可以了！
</p>

```sql
GRANT "CONNECT" TO "BBB" ;
GRANT "RESOURCE" TO "BBB" ;
GRANT "DBA" TO "BBB" ;
GRANT "UMEC_ROLE" TO "BBB" ;
```
