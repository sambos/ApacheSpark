## Claudera Impala
- A distributed SQL query engine for Hadoop

#### Impala commands 
Locate the daemon server from cloudera manager or hortonworks 
> impala-shell -k -i \<impala-daemon-server-name\>


#### Comparing Impala/Hive/SparkSQL
https://blog.cloudera.com/blog/2016/02/new-sql-benchmarks-apache-impala-incubating-2-3-uniquely-delivers-analytic-database-performance/

#### Connecting to Impala/Hive from your workstation

###### using LDAP/Kerberos Authentication
Here is the good article explaining the concepts - 
[Impala Authentication with LDAP Kerberos](http://blog.cloudera.com/blog/2014/10/new-in-cdh-5-2-impala-authentication-with-ldap-and-kerberos/)

###### Create Keytab file
```shell
$ktutil
$ktutil: addent -password -p username@REALM -k 1 -e rc4-hmac
$ktutil:wkt /home/name/majeed.keytab
$ktutil: quit

```
###### Test connection from Impala Shell
```shell
## put data to hdfs
hdfs dfs -copyFromLocal city.txt /user/name/input

## change permissions
hdfs dfs -chmod 777 /user/name/input/city.txt

$ impala-shell -k -i <impala-daemon-host>
[impala-daemon-host:21000] > show schemas;
Query: show schemas
+----------+
| name     |
+----------+
| default  |
+----------+


CREATE EXTERNAL TABLE IF NOT EXISTS sampledb.citytable
(
    st STRING COMMENT 'State', 
    ct STRING COMMENT 'City', 
    po STRING COMMENT 'Population'
) 
COMMENT 'Original Table' 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
LOCATION '/user/name/input/city.txt';

-- you can aswell load using
load data inpath '/user/name/input/city.txt' into table sampledb.citytable

```
###### dependecies
Following dependences are required

```xml
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-jdbc</artifactId>
			<version>0.14.0</version>
			<exclusions>
				<exclusion>
					<artifactId>jetty-all</artifactId>
					<groupId>org.eclipse.jetty.aggregate</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-common</artifactId>
			<version>2.7.2</version>
		</dependency>
```


###### Example connection from workstation
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
    	System.setProperty("java.security.krb5.realm", "<your-domain>");
    		   
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

###### Steps to execute
```sh
javac  -classpath /opt/cloudera/parcels/CDH-5.5.2-1.cdh5.5.2.p0.4/jars/hadoop-common-2.6.0-cdh5.5.2.jar HiveImpalaJdbcDriver.java
export CLASSPATH=${CLASSPATH}:/opt/cloudera/parcels/CDH-5.5.2-1.cdh5.5.2.p0.4/lib/hive/lib/*:/opt/cloudera/parcels/CDH-5.5.2-1.cdh5.5.2.p0.4/jars/*
java HiveImpalaJdbcDriver 
```

