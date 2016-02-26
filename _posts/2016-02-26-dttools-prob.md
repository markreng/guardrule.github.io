--- 
layout: post 
title: 个性化变形方式处理 
--- 

为处理测试中心提出的新变形需求，进行了个性化变形操作，其中遇到多个问题，现记录备忘。 
需要一张表。 
开发中心要对这张表做部分变形，采用旧变形方式，存在阴阳表: 
    OPEN VCUR; 
    LOOP 
    BEGIN 
        FETCH VCUR BULK COLLECT INTO VROWID,VVALUE LIMIT INT_BULKCOUNT; 

其中发现两个问题： 
    1.虽然dtrans用户为dba，但是存储过程无法调用cmis3用户数据。需要grant all on schema.tablename to dtrans; 
    2.执行阴阳表会发生数据部分未变形问题，全表200行数据，小，采用一次性commit； 
    3.在plsql端执行发现没有报错，因为set serverout on没有开，打开后发现各种trigger,index,constraint，所以全干掉。 

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


谢谢！
 	任岗
——————————————
数据中心（上海） 系统三部
电话 : 021-28989782
E-mail : gren@dccsh.icbc.com.cn


---------------------------------------------------------------------     
    此邮件信息仅供收件人查阅，所含任何评论、陈述或数据仅供收件人参考，若有改动，恕可能不另行通知。未经中国工商银行书面许可，请勿披露、复制、转载此邮件信息。任何第三方均不得查阅或使用此邮件信息。若您误收到本邮件，敬请及时通知发件人，并将邮件从您系统中彻底删除。发件人及中国工商银行均不对因邮件可能引发的损失负责。
This message is intended only for use of the addressees and any comment, statement or data contained herein is for the reference of the receivers only. Notification may not be sent for any revising related. Please do not disclose, copy, or distribute this e-mail without ICBC written permission. Any third party shall not read or use the content of this e-mail. If you receive this e-mail in error, please notify the sender immediately and delete this e-mail from your computer system completely.The sender and ICBC are not responsible for the loss caused possibly by e-mail.
