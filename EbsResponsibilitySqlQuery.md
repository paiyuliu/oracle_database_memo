EBS Responsibility SQL Query
===

###### tags: `新手` `EBS` ``

剛接觸Oracle ERP - EBS ，實在是太奇怪的軟體了！整個架構跟以前系統開發完全不一樣
權限控管那些都是另外一個架構，在這邊將網路上熱心開發者所列出常用的sql，對我來說就是將我所面對的user有哪些responsibility及對應的function和concurrment！

[原文連結](https://technology.amis.nl/2008/11/11/some-sql-scripts-for-analyzing-oracle-ebs/#MenusListing)

[TOC]

While working in Oracle eBS it can sometimes be very helpful to have a quick overview of how and/or what is setup. Sometimes it is enough to just use what was made available to you and by clicking through the available setup screens you will simply find what you are looking for. However, in some cases you end up not having the right responsibilities to see these details or you just want to prevent scrolling and clicking through these windows or combine details from different screens.

In addition to the diagnostic scripts that Oracle provided there are various other scripts available that can be very useful from case to case.
I have posted some of the scripts I have been using in the past. If you have some good similar script(s) available that you have been using and willing to share, please feel free to do so.
I am sure you’ll make a lot of people happy.

1. Responsibilities Listing(列出哪些權限)

    ```sql
    /*
    Purpose/Description:
    Retrieve a list of all responsibilities.
    Parameters
    None
    */
    SELECT (  SELECT APPLICATION_SHORT_NAME
              FROM APPS.FND_APPLICATION FA
              WHERE FA.APPLICATION_ID = FRT.APPLICATION_ID
           ) APPLICATION,
           FRT.RESPONSIBILITY_ID,
           FRT.RESPONSIBILITY_NAME
    FROM APPS.FND_RESPONSIBILITY_TL FRT
    ;
    
    ```

2. Menus Listing(列出那些Menu)

    ```sql
    /*
    Purpose/Description:
    To see the Menus associated with a given responsibility
    Parameters
    responsibility_id that you can retrieve from query nr 1 (Responsibilities Listing)
    */
    SELECT DISTINCT A.RESPONSIBILITY_NAME, C.USER_MENU_NAME
      FROM APPS.FND_RESPONSIBILITY_TL A,
           APPS.FND_RESPONSIBILITY    B,
           APPS.FND_MENUS_TL          C,
           APPS.FND_MENUS             D,
           APPS.FND_APPLICATION_TL    E,
           APPS.FND_APPLICATION       F
     WHERE A.RESPONSIBILITY_ID(+) = B.RESPONSIBILITY_ID
          --AND a.responsibility_id = 50103
       AND C.USER_MENU_NAME LIKE '%UM_PO%'
       AND B.MENU_ID = C.MENU_ID
       AND B.MENU_ID = D.MENU_ID
       AND E.APPLICATION_ID = F.APPLICATION_ID
       AND F.APPLICATION_ID = B.APPLICATION_ID
       AND A.LANGUAGE = 'US'
       ;

    ```

3. Submenu And Function Listing

    ```sql
    /**
    Purpose/Description:
    By using this query you can check function and submenus attached to a specific menu
    Parameters
    User_menu_name that you can get by running query 2 (Menu Listing)
    */
    
    SELECT A.USER_MENU_NAME, C.PROMPT, C.DESCRIPTION
    FROM APPS.FND_MENUS_TL A, APPS.FND_MENU_ENTRIES_TL C
    WHERE A.MENU_ID = C.MENU_ID
    AND A.USER_MENU_NAME LIKE '%PO%';
    /*   AND a.user_menu_name = 'Navigator Menu - System Administrator GUI'*/
    
    ```

4. User and Assigned Responsibility Listing(看user有哪些 responsibility )

    ```sql
    /*
    Purpose/Description:
    You can use this query to check responsibilities assigned to users.
    Parameters None
    */
    SELECT UNIQUE U.USER_ID,
           SUBSTR(U.USER_NAME, 1, 30) USER_NAME,
           SUBSTR(R.RESPONSIBILITY_NAME, 1, 60) RESPONSIBLITY,
           SUBSTR(A.APPLICATION_NAME, 1, 50) APPLICATION
      FROM APPS.FND_USER              U,
           APPS.FND_USER_RESP_GROUPS  G,
           APPS.FND_APPLICATION_TL    A,
           APPS.FND_RESPONSIBILITY_TL R
     WHERE G.USER_ID(+) = U.USER_ID
       AND G.RESPONSIBILITY_APPLICATION_ID = A.APPLICATION_ID
       AND A.APPLICATION_ID = R.APPLICATION_ID
       AND G.RESPONSIBILITY_ID = R.RESPONSIBILITY_ID
     ORDER BY SUBSTR(USER_NAME, 1, 30),
              SUBSTR(A.APPLICATION_NAME, 1, 50),
              SUBSTR(R.RESPONSIBILITY_NAME, 1, 60)
    ;
    
    ```


5. Responsibility and assigned request group listing(Resposibility與Request群組的對應)

    ```sql
    /**
    Purpose/Description:
    To find responsibility and assigned request groups.
    Every responsibility contains a request group (The request group is basis of submitting requests)
    Parameters None
    */
    SELECT RESPONSIBILITY_NAME RESPONSIBILITY,
           REQUEST_GROUP_NAME,
           FRG.DESCRIPTION
    FROM APPS.FND_REQUEST_GROUPS FRG, APPS.FND_RESPONSIBILITY_VL FRV
    WHERE FRV.REQUEST_GROUP_ID = FRG.REQUEST_GROUP_ID
    ORDER BY RESPONSIBILITY_NAME
    ;
    
    ```

6. Profile option with modification date and user

    ```sql
    /**
    Purpose/Description:
    Query that can be used to audit profile options.
    Parameters
    None
    **/
    SELECT T.USER_PROFILE_OPTION_NAME,
           PROFILE_OPTION_VALUE,
           V.CREATION_DATE,
           V.LAST_UPDATE_DATE,
           V.CREATION_DATE - V.LAST_UPDATE_DATE "Change Date",
           (SELECT UNIQUE USER_NAME
              FROM APPS.FND_USER
             WHERE USER_ID = V.CREATED_BY) "Created By",
           (SELECT USER_NAME
              FROM APPS.FND_USER
             WHERE USER_ID = V.LAST_UPDATED_BY) "Last Update By"
      FROM APPS.FND_PROFILE_OPTIONS       O,
           APPS.FND_PROFILE_OPTION_VALUES V,
           APPS.FND_PROFILE_OPTIONS_TL    T
     WHERE O.PROFILE_OPTION_ID = V.PROFILE_OPTION_ID
       AND O.APPLICATION_ID = V.APPLICATION_ID
       AND START_DATE_ACTIVE <= SYSDATE
       AND NVL(END_DATE_ACTIVE, SYSDATE) >= SYSDATE
       AND O.PROFILE_OPTION_NAME = T.PROFILE_OPTION_NAME
       AND LEVEL_ID = 10001
       AND T.LANGUAGE IN (SELECT LANGUAGE_CODE
                            FROM APPS.FND_LANGUAGES
                           WHERE INSTALLED_FLAG = 'B'
                          UNION
                          SELECT NLS_LANGUAGE
                            FROM APPS.FND_LANGUAGES
                           WHERE INSTALLED_FLAG = 'B')
     ORDER BY USER_PROFILE_OPTION_NAME
    ;
    
    ```

7. Forms personalization Listing(看Form的個人化)
<p>personalization指得是Form在另外加條件，而不寫在fmd中</p>

    ```sql
    /**
    Purpose/Description:
    To get modified profile options.
    Personalization is a feature available in 11.5.10.X.
    Parameters None
    */
    -- ALTER SESSION SET nls_language = 'AMERICAN';
    SELECT FFFT.USER_FUNCTION_NAME "User Form Name",
           FFCR.SEQUENCE,
           FFCR.DESCRIPTION,
           FFCR.RULE_TYPE,
           FFCR.ENABLED,
           FFCR.TRIGGER_EVENT,
           FFCR.TRIGGER_OBJECT,
           FFCR.CONDITION,
           FFCR.FIRE_IN_ENTER_QUERY,
           (SELECT USER_NAME FROM FND_USER FU WHERE FU.USER_ID = FFCR.CREATED_BY) "Created By"
      FROM FND_FORM_CUSTOM_RULES FFCR, FND_FORM_FUNCTIONS_VL FFFT
     WHERE FFCR.ID = FFFT.FUNCTION_ID
     ORDER BY 1
     ;
    
    ```

8. Patch Level Listing
    ```sql
    /**
    Purpose/Description:
    Query that can be used to view the patch level status of all modules
    Parameters None
    */
    SELECT A.APPLICATION_NAME,
           DECODE(B.STATUS, 'I', 'Installed'
                          , 'S', 'Shared', 'N/A') STATUS,
           PATCH_LEVEL
    FROM APPS.FND_APPLICATION_VL A
       , APPS.FND_PRODUCT_INSTALLATIONS B
    WHERE A.APPLICATION_ID = B.APPLICATION_ID
    ;
    
    ```

9. Request attached to responsibility listing(看Responsibility與Request的對照)

    ```sql
    /**
    Purpose/Description:
    To see all requests attached to a responsibility
    Parameters None
    */
    SELECT
        responsibility_name
    ,   frg.request_group_name
    ,   fcpv.user_concurrent_program_name
    ,   fcpv.description
    FROM
        apps.fnd_request_groups frg
    ,   apps.fnd_request_group_units frgu
    ,   apps.fnd_concurrent_programs_vl fcpv
    ,   apps.fnd_responsibility_vl frv
    WHERE
        frgu.request_unit_type = 'P'
    AND frgu.request_group_id = frg.request_group_id
    AND frgu.request_unit_id = fcpv.concurrent_program_id
    AND frv.request_group_id = frg.request_group_id
    ORDER BY responsibility_name
    ;
    
    ```

10. Request listing application wise

    ```sql
    /**
    Purpose/Description:
    View all request types application wise
    Parameters None*/
    ALTER SESSION SET nls_language = 'AMERICAN';
    SELECT
      fa.application_short_name
    , fcpv.user_concurrent_program_name
    , description
    , DECODE (fcpv.execution_method_code
     ,'B', 'Request Set Stage Function'
     ,'Q', 'SQL*Plus'
     ,'H', 'Host'
     ,'L', 'SQL*Loader'
     ,'A', 'Spawned'
     ,'I', 'PL/SQL Stored Procedure'
     ,'P', 'Oracle Reports'
     ,'S', 'Immediate'
     ,fcpv.execution_method_code) exe_method
    ,   output_file_type
    ,   program_type
    ,   printer_name
    ,   minimum_width
    ,   minimum_length
    ,   concurrent_program_name
    ,   concurrent_program_id
    FROM apps.fnd_concurrent_programs_vl fcpv
    ,    apps.fnd_application fa
    WHERE fcpv.application_id = fa.application_id
    ORDER BY DESCRIPTION
    ;
    
    ```

11. Count Reports per module(每個模組有幾個Report)

    ```sql
    /**
    Purpose/Description:
    To Count Reports
    Parameters None
    */
    SELECT FA.APPLICATION_SHORT_NAME,
           DECODE(FCPV.EXECUTION_METHOD_CODE,
                  'B','Request Set Stage Function',
                  'Q','SQL*Plus',
                  'H','Host',
                  'L','SQL*Loader',
                  'A','Spawned',
                  'I','PL/SQL Stored Procedure',
                  'P','Oracle Reports',
                  'S','Immediate',
                  FCPV.EXECUTION_METHOD_CODE) EXE_METHOD,
           COUNT(CONCURRENT_PROGRAM_ID) COUNT
      FROM APPS.FND_CONCURRENT_PROGRAMS_VL FCPV, APPS.FND_APPLICATION FA
     WHERE FCPV.APPLICATION_ID = FA.APPLICATION_ID
     GROUP BY FA.APPLICATION_SHORT_NAME, FCPV.EXECUTION_METHOD_CODE
     ORDER BY 1
     ;
    
    ```

12. Request Status Listing

    ```sql
    /**
    Purpose/Description:
    This query returns report/request processing time
    Parameters None
    */
    SELECT
      f.request_id
    ,   pt.user_concurrent_program_name user_concurrent_program_name
    ,   f.actual_start_date actual_start_date
    ,   f.actual_completion_date actual_completion_date
    , floor(((f.actual_completion_date-f.actual_start_date)*24*60*60)/3600)
     || ' HOURS ' ||
      floor((((f.actual_completion_date-f.actual_start_date)*24*60*60) -
      floor(((f.actual_completion_date-f.actual_start_date)*24*60*60)/3600)*3600)/60)
     || ' MINUTES ' ||
      round((((f.actual_completion_date-f.actual_start_date)*24*60*60) -
      floor(((f.actual_completion_date-f.actual_start_date)*24*60*60)/3600)*3600 -
     (floor((((f.actual_completion_date-f.actual_start_date)*24*60*60) -
      floor(((f.actual_completion_date-f.actual_start_date)*24*60*60)/3600)*3600)/60)*60) ))
     || ' SECS ' time_difference
    , DECODE(p.concurrent_program_name
     ,'ALECDC'
     ,p.concurrent_program_name||'['||
     f.description||']'
     ,p.concurrent_program_name) concurrent_program_name
    , decode(f.phase_code
     ,'R','Running'
     ,'C','Complete'
     ,f.phase_code) Phase
    , f.status_code
    FROM
        apps.fnd_concurrent_programs p
    ,   apps.fnd_concurrent_programs_tl pt
    ,   apps.fnd_concurrent_requests f
    WHERE
      f.concurrent_program_id = p.concurrent_program_id
    AND f.program_application_id = p.application_id
    AND f.concurrent_program_id = pt.concurrent_program_id
    AND f.program_application_id = pt.application_id
    AND pt.language = USERENV('Lang')
    AND f.actual_start_date is not null
    ORDER by f.actual_completion_date-f.actual_start_date desc
    ;
    
    ```

13. User and responsibility listing

    ```sql
    /**
    Purpose/Description:
    Check responsibilities assigned to users
    Parameters None
    */
    SELECT UNIQUE U.USER_ID,
           SUBSTR(U.USER_NAME, 1, 30) USER_NAME,
           SUBSTR(R.RESPONSIBILITY_NAME, 1, 60) RESPONSIBLITY,
           SUBSTR(A.APPLICATION_NAME, 1, 50) APPLICATION
      FROM APPS.FND_USER              U,
           APPS.FND_USER_RESP_GROUPS  G,
           APPS.FND_APPLICATION_TL    A,
           APPS.FND_RESPONSIBILITY_TL R
     WHERE G.USER_ID(+) = U.USER_ID
       AND G.RESPONSIBILITY_APPLICATION_ID = A.APPLICATION_ID
       AND A.APPLICATION_ID = R.APPLICATION_ID
       AND G.RESPONSIBILITY_ID = R.RESPONSIBILITY_ID
    -- AND a.application_name like '%Order Man%'
     ORDER BY SUBSTR(USER_NAME, 1, 30),
              SUBSTR(A.APPLICATION_NAME, 1, 50),
              SUBSTR(R.RESPONSIBILITY_NAME, 1, 60)
    ;
    
    ```

14. Applied Patch Listing

    ```sql
    /**
    Purpose/Description:
    Check Current Applied Patches
    Parameters None
    */
    SELECT
      patch_name
    ,   patch_type
    ,   maint_pack_level
    ,   creation_date
    FROM applsys.ad_applied_patches
    ORDER BY creation_date DESC
    ;
    
    ```

[原文連結二](https://community.oracle.com/thread/3687782)
R12 SQL for responsabilities and menu
Hello
I would like to ask you if you can share with me any SQL to get the user, responsabilities and menus.
I have the following example.
Thanks

It shows what is current and has columns on the right showing what is ended or excluded.

--Matrix of Application, Responsibility, Menu, Prompt, Function, SubMenu, SubFunction
--that shows resposibility end date and identifies exclusions

然後是很可怕的sql

```sql

select second.application_id        "App ID"
     , second.application_name      "App Name"
     , second.responsibility_id     "Resp ID"
     , second.responsibility_name   "Responsibility"
     , second.menu_id               "Menu ID"
     , second.user_menu_name        "Main Menu Name"
     , second.entry_sequence        "Seq"
     , second.prompt                "Prompt"
     , second.function_id           "Function ID"
     , second.user_function_name    "Function"
     , second.func_descrip          "Function Descrip"
     , second.sub_menu_id           "SubMenu ID"
     , second.sub_menu_name         "SubMenu Name"
     , second.sub_seq               "Sub Seq"
     , second.sub_prompt            "SubPrompt"
     , second.sub_func_id           "SubFunction ID"
     , second.sub_func              "SubFunction"
     , second.sub_func_descrip      "SubFunction Descrip"
     , second.sub_sub_menu_id       "Sub-SubMenu ID"    
     , second.grant_flag            "Grant Flag"
     , second.resp_end_date         "Resp End Date"
     , decode(exc.rule_type, 'F', (select 'Ex F: ' || exc.action_id
                                   from fnd_form_functions_vl  fnc
                                   where fnc.function_id      = exc.action_id
                                   and   second.function_id   = exc.action_id
                                  )                            
               ) Excluded_Function
     , decode(exc.rule_type, 'F', (select 'Ex SF: ' || exc.action_id
                                   from fnd_form_functions_vl  fnc
                                   where fnc.function_id      = exc.action_id
                                   and   second.sub_func_id   = exc.action_id
                                  )                            
               ) Excluded_Sub_Function                 
     , decode(exc.rule_type, 'M', (select 'Ex M: ' || exc.action_id
                                   from fnd_form_functions_vl  fnc
                                   where fnc.function_id      = exc.action_id
                                   and   second.menu_id       = exc.action_id
                                  )                            
               ) Excluded_Menu  
     , decode(exc.rule_type, 'M', (select 'Ex SM: ' || exc.action_id
                                   from fnd_form_functions_vl  fnc
                                   where fnc.function_id      = exc.action_id
                                   and   second.sub_menu_id   = exc.action_id
                                  )                            
               ) Excluded_Sub_Menu          
     , decode(exc.rule_type, 'M', (select 'Ex SSM: ' || exc.action_id
                                   from fnd_form_functions_vl  fnc
                                   where fnc.function_id          = exc.action_id
                                   and   second.sub_sub_menu_id   = exc.action_id
                                  )                            
               ) Excluded_Sub_Sub_Menu

from
        (select first.application_id      
              , first.application_name     
              , first.responsibility_id    
              , first.responsibility_name  
              , first.end_date              as resp_end_date
              , first.menu_id               
              , first.user_menu_name       
              , first.entry_sequence       
              , first.prompt               
              , first.function_id          
              , ffft.user_function_name    
              , ffft.description            as func_descrip
              , first.sub_menu_id          
              , fmv2.user_menu_name         as sub_menu_name
              , fme2.entry_sequence         as sub_seq
              , fmet2.prompt                as sub_prompt
              , fme2.function_id            as sub_func_id 
              , ffft2.user_function_name    as sub_func
              , ffft2.description           as sub_func_descrip
              , fme2.sub_menu_id            as sub_sub_menu_id 
              , first.grant_flag          

        from
            (
                select fat.application_id       
                     , fat.application_name     
                     , fr.responsibility_id     
                     , frt.responsibility_name
                     , fr.end_date 
                     , fr.menu_id               
                     , fmv.user_menu_name      
                     , fme.entry_sequence       
                     , fmet.prompt             
                     , fme.sub_menu_id          
                     , fme.function_id          
                     , fme.grant_flag           
                    
                from apps.fnd_application_tl        fat  
                   , apps.fnd_responsibility        fr
                   , apps.fnd_menus_vl              fmv
                   , apps.fnd_responsibility_tl     frt
                   , apps.fnd_menu_entries          fme
                   , apps.fnd_menu_entries_tl       fmet

                --joins and constant selection
                where fat.application_id   = fr.application_id(+)
                and   fr.menu_id           = fmv.menu_id(+)
                and   fr.responsibility_id = frt.responsibility_id(+)  
                and   fr.menu_id           = fme.menu_id(+)
                and   fme.menu_id          = fmet.menu_id(+)
                and   fme.entry_sequence   = fmet.entry_sequence(+)  
                and   fmet.language        = 'US'


                --------------------------------------
                -- add specific selection criteria  --
                --------------------------------------
                --and   fat.application_id = 8401   --for DEVL  19080 rows
                --and fr.responsibility_id = 51856
                and   fat.application_id = &AppID


                order by 1,2,3,4,5,6,7,8,9,10,11,12

           )                                 first     ---for application, responsibility and main menu info
         
            , apps.fnd_menus_vl              fmv2      ---for submenu info
            , apps.fnd_menu_entries          fme2
            , apps.fnd_menu_entries_tl       fmet2

            , apps.fnd_form_functions_tl     ffft      ---for function info          
            , apps.fnd_form_functions_tl     ffft2     ---for subfunction info
           
        --left outer joins keep original records and add any sub menu and function info
        where first.function_id    = ffft.function_id(+)  
        and   first.sub_menu_id    = fmv2.menu_id(+)
        and   first.sub_menu_id    = fme2.menu_id(+)
        and   fme2.menu_id         = fmet2.menu_id(+)
        and   fme2.entry_sequence  = fmet2.entry_sequence(+)
        and   fme2.function_id     = ffft2.function_id(+)

        order by 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21

        )                                     second     -- adds any sub menu and function info

        left outer join apps.fnd_resp_functions          exc       ---for exclusions
        on (    second.application_id        = exc.application_id
            and second.responsibility_id     = exc.responsibility_id
            and (   second.function_id       = exc.action_id
                 or second.sub_func_id       = exc.action_id
                 or second.menu_id           = exc.action_id
                 or second.sub_menu_id       = exc.action_id
                 or second.sub_sub_menu_id   = exc.action_id
                )
           )
order by 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21;

```
