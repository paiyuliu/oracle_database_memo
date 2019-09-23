Oracle 欄位註記(Column Comment)
===
[原文網址](https://blog.xuite.net/fenctang/twblog/106972954-[Oracle+SQL]+為資料欄位標示註釋，comment+指令)

##### 發現所有同仁都沒針對Table欄位寫註記(不是幾乎，是全部)
##### 先記錄下來

```sql
COMMENT ON COLUMN [schema].table_name.field_name is '備註'
```
###### Example:新增Table Field 備註
```sql
COMMENT ON COLUMN rec_master.m_plant IS '廠別'
```
###### 查詢
   ```sql
    SELECT column_name,comments FROM user_col_comments WHERE table_name=UPPER('rec_master')
    /*
    COLUMN_NAME COMMENTS
    ------------------------- ------------------
    M_PLANT               廠別
    */
   ```
###### 刪除
   ```sql
    comment on column [schema].table_name.field_name is ' '
   ```