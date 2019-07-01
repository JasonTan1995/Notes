## Oracle 如何插入特殊字符: & 或者 @

### 1. 前言
***
最近在一次导数据中,`Oracle`会弹出一个输入框,说是要为某个变量进行赋值.然后搜索了一会发现`Oracle`会把`& nbsp;`当成自定义变量了,那需要怎么样解决呢?

### 2. 解决办法
***
#### 2.1  `Set define off`

方法一：在要插入的SQL语句前加上Set define off;与原SQL语句一起批量执行

这个是Oracle里面用来识别自定义变量的设置.

```sql
SET DEFINE OFF;
Insert into FORM_LKP (FORM_ID,FORM_NAME,STATUS,HANDLER,DEPT_ID,DESCRIPTION,KEYWORD,CCC,VERSION,WORKING_PARTY,SEQ,SUPPORT_TEMPLATE_IMPORT,FORM_NAME_CN,PRODUCT_TYPE,LAUNCH_DATE,T_C_URL,COLLECTION_STATEMENT_URL)
values (1037,'Broadband 300MB','ACTIVE',null,'1','this is test form',null,'CX52',1,'BBISALE',500,'N','Broadband 300MB','BBI',to_date('21-6月-19','DD-MON-RR'),null,null);
```

#### 2.2 转义

```sql
'&'||'nbsp;&'||'nbsp;'
```
对字符进行转义

### 参考资料

[Oracle中如何插入特殊字符：& 和 ' (多种解决方案)](http://www.blogjava.net/pengpenglin/archive/2008/01/16/175689.html)
