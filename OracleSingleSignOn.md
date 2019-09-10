Oracle 登入的寫法
===
[原文網址](https://www.javaworld.com.tw/jute/post/view?bid=35&id=97119&sty=1&tpg=2&)

##### 之後應該有使用者登入的需求
##### 先記錄下來
##### 重點應該是 oracle.apps.fnd.security.AolSecurity() 那個物件方法
##### .net 應該是一樣的

   ```java
    %@ page contentType="text/html;charset=Big5" import="java.util.*,java.sql.*"%> 
    
    <%
    String userId = "";
    String encryptPassword = "";
    String decryptPassword = "";
    String userName = "";
    String sql = "";
    
    //連結Oracle DB
    Class.forName("oracle.jdbc.driver.OracleDriver").newInstance();
    Connection con = DriverManager.getConnection("jdbc:oracle:thin:@192.168.1.1:1521:PROD", "ERP DB帳號", "ERP DB 密碼");
    Statement stmt = con.createStatement();
    
    //取得使用者的名字
    userName = request.getParameter("name");
    
    //由Oracle ERP 中的 fnd_user 取得該使用者加密後的密碼
    Sql ="select * from fnd_user where description like '%"+userName+"%'";
    ResultSet rs = stmt.executeQuery(sql);
    
    while (rs.next())
    {
    userId = rs.getString("USER_NAME");
    encryptPassword = rs.getString("ENCRYPTED_USER_PASSWORD");
    }
    
    //透過Oracle ERP 的API 將加密後的密碼轉成未加密的密碼
    oracle.apps.fnd.security.AolSecurity aolsec=new oracle.apps.fnd.security.AolSecurity();
    decryptPassword = aolsec.decrypt("APPS",encryptPassword);
    
    //將帳號密碼以Hidden field 的方式填入的Form 再由JavaScript來Submit達到登入ERP的功能
    %>
    <FORM NAME="Logon0" ACTION="http://erp:8001/pls/PROD/oraclemypage.home" METHOD="POST" TARGET="_blank"> 
    <INPUT TYPE="hidden" NAME="i_1" VALUE="<%=userId%>">
    <INPUT TYPE="hidden" NAME="i_2" VALUE="<%=decryptPassword%>">
    <INPUT TYPE="hidden" NAME="rmode" VALUE="2">
    <INPUT TYPE="hidden" NAME="home_url" VALUE="">
    </FORM>
    <SCRIPT LANGUAGE="javascript">
        document.Logon0.submit();
    </SCRIPT>

   ```