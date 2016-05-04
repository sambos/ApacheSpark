## Claudera Impala
- A distributed SQL query engine for Hadoop

#### Impala commands 
Locate the daemon server from cloudera manager or hortonworks 
impala-shell -k -i <impala-daemon-server-name>


#### Comparing Impala/Hive/SparkSQL
https://blog.cloudera.com/blog/2016/02/new-sql-benchmarks-apache-impala-incubating-2-3-uniquely-delivers-analytic-database-performance/

#### Connecting to Impala/Hive from your workstation

```java
import java.io.File;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import org.apache.hadoop.security.UserGroupInformation;

public class HiveImpalaJdbcDriver {
	static String JDBCDriver = "org.apache.hive.jdbc.HiveDriver";
    //private static final String CONNECTION_URL = "jdbc:hive2://<server>:10000/default;principal=hive/<server>@domain";
	 private static final String CONNECTION_URL = "jdbc:hive2://<server>:21050/default;principal=impala/<server>@domain";    
    public static void main(String[] args) {
    		System.out.println("from HiveImpalaJdbcDriver");
    		
        String realm = "UNITOPR.UNITINT.TEST.STATEFARM.ORG";
    		System.setProperty("java.security.krb5.realm", realm);
    		   
        System.setProperty("java.security.krb5.kdc", realm + ":88");   //KDC-Port 88        
        String keytabpath = "./src/main/resources/majeed.keytab";
        keytabpath = new File(keytabpath).getAbsolutePath();
    		System.out.println("keytab path ::" + keytabpath);
    		
        org.apache.hadoop.conf.Configuration conf = new org.apache.hadoop.conf.Configuration();
        conf.set("hadoop.security.authentication", "Kerberos");
        UserGroupInformation.setConfiguration(conf);

            try {
            	
                    UserGroupInformation.loginUserFromKeytab("user@domain", keytabpath);
            } catch (IOException e) {
                    e.printStackTrace();
            }
            try {
                    Class.forName(JDBCDriver);
                    System.out.println("Connecting to server...");
                    Connection con = DriverManager.getConnection(CONNECTION_URL);
                    String query = "Select * from schema.table";
                    Statement stmt = con.createStatement();
                    System.out.println("Executing Query...");
                    ResultSet rs = stmt.executeQuery(query);
                    while (rs.next()) {
                            System.out.println(rs.getString(1) + "," + rs.getString(2));

                    }

            } catch (ClassNotFoundException e) {
                    e.printStackTrace();
            } catch (SQLException e) {
                    e.printStackTrace();
            }

    }
}


```
