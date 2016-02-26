--- 
layout: post
catagority: [数据变形]
tag: [数据变形]
title: 个性化数据变形问题处理 
--- 
### 问题来源 ###
为处理测试中心提出的新变形需求，进行了个性化变形操作，其中遇到多个问题，现记录备忘。 
需要一张表。 
开发中心要对这张表做部分变形，采用旧变形方式，存在阴阳表: 

    OPEN VCUR; 
    LOOP 
    BEGIN 
        FETCH VCUR BULK COLLECT INTO VROWID,VVALUE LIMIT INT_BULKCOUNT; 

其中发现两个问题：

    1. 虽然dtrans用户为dba，但是存储过程无法调用cmis3用户数据。需要grant all on schema.tablename to dtrans; 
    2. 执行阴阳表会发生数据部分未变形问题，全表200行数据，小，采用一次性commit； 
    3. 在plsql端执行发现没有报错，因为set serverout on没有开，打开后发现各种trigger,index,constraint，所以全干掉。 

完成的脚本如下: 

        DECLARE 
          N NUMBER; 
        BEGIN
          pack_dtrans_func.proc_init_para('1510201455055067431'); 
          pack_dtrans_func.proc_init_id_table; 
          pack_dtrans_func.proc_init_key_passwd_table; 
          SELECT COUNT(*) INTO N FROM CMIS3.TA200265 WHERE TA200265003='00020' or TA200265003='00021'; 
          DBMS_OUTPUT.PUT_LINE('ROWNUM = '||N); 
          UPDATE CMIS3.TA200265 SET TA200265005 = PACK_DTRANS_FUNC.FUNC_trans_id(TA200265005) WHERE TA200265003='00020' or TA200265003='00021'; 
          COMMIT; 
        EXCEPTION WHEN OTHERS THEN 
                 DBMS_OUTPUT.PUT_LINE(SQLERRM); 
        END; 
        / 
        
        DECLARE 
          N NUMBER; 
        BEGIN 
            pack_dtrans_func.proc_init_para('1510201455055067431'); 
            pack_dtrans_func.proc_init_id_table; 
            pack_dtrans_func.proc_init_key_passwd_table; 
            SELECT COUNT(*) INTO N FROM CMIS3.TA200265 where TA200265003<>'00020' and TA200265003<>'00021'; 
            DBMS_OUTPUT.PUT_LINE('ROWNUM = '||N); 
            UPDATE CMIS3.TA200265 SET TA200265021 = PACK_DTRANS_FUNC.FUNC_trans_id(TA200265021) where TA200265003<>'00020' and TA200265003<>'00021'; 
            COMMIT; 
        EXCEPTION WHEN OTHERS THEN 
                ROLLBACK; 
                 DBMS_OUTPUT.PUT_LINE(SQLERRM); 
        END; 
        / 

后续要研究下如何合理，进度可控，路径优化的实现“数据变形工具”的闭包。 

--EOF--
