SQL Monitoring Extension
====================================

## Use Case ##
This extension can be used to query an SQL Database and the resulting values can be used as metrics on AppDynamics.
The connection to the database is established through a JDBC connect and you will have to use a "connector" JAR file in order to have the extension connect and query the database.

The metrics reported by the extension can be modified as per the user's requirements. This extension can be used to query and pull metrics from any SQL based database.

### Prerequisites ###
1. This extension requires a AppDynamics Java Machine Agent installed and running.
2. This extension requires that the user provide their own Jar file in order to connect to the Database. 

## Installation ##

1. To build from the source, run "mvn clean install" and find the SQLMonitor.zip file in the "target" folder.
   You can also download the SQLMonitor.zip from [AppDynamics Exchange][].
2. Unzip as "SQLMonitor" and copy the "SQLMonitor" directory to `<MACHINE_AGENT_HOME>/monitors`.

    Note: You will need to provide your own JDBC driver for the database you want to connect to. Put the driver JAR file in the same directory and add it to the classpath element in the
monitor.xml file.!

### Example ###
```
<java-task>
    <!-- Use regular classpath foo.jar;bar.jar -->
    <!-- append JDBC driver jar -->
    <classpath>sql-monitoring-extension.jar:Jar-File-For_Your-DB.jar</classpath>
    <impl-class>com.appdynamics.monitors.sql.SQLMonitor</impl-class>
</java-task>
```
3. Edit the config.yaml file. An example config.yaml file follows these installation instructions.
4. Restart the Machine Agent.


## Configuration ##


**Note** : Please make sure to not use tab (\t) while editing yaml files. You may want to validate the yaml file using a yaml validator http://yamllint.com/

You will have to Configure the SQL server instances by editing the config.yaml file in `<MACHINE_AGENT_HOME>/monitors/SQLMonitor/`. 
The information provided in this file will be used to connect and query the database. 
You can find a sample config.yaml file below.

   ```
   # Make sure the metric prefix ends with a |
   #This will create this metric in all the tiers, under this path.
   #metricPrefix: "Custom Metrics|SQL|"
   #This will create it in specific Tier. Replace <ComponentID> with TierID
   metricPrefix: "Server|Component:<ComponentID>|Custom Metrics|SQL|"
   
   
   dbServers:
       - displayName: "Instance1"
         connectionUrl: ""
         driver: ""
   
         connectionProperties:
           - user: ""
           - password: ""
   
         #Needs to be used in conjunction with `encryptionKey`. Please read the extension documentation to generate encrypted password
         #encryptedPassword: ""
   
         #Needs to be used in conjunction with `encryptedPassword`. Please read the extension documentation to generate encrypted password
         #encryptionKey: "welcome"
   
         # Replaces characters in metric name with the specified characters.
         # "replace" takes any regular expression
         # "replaceWith" takes the string to replace the matched characters
   
         metricCharacterReplacer:
           - replace: "%"
             replaceWith: ""
           - replace: ","
             replaceWith: "-"
   
   
         queries:
           - displayName: "Active Events"
             queryStmt: "Select NODE_NAME, EVENT_CODE, EVENT_ID, EVENT_POSTED_COUNT from Active_events"
             columns:
               - name: "NODE_NAME"
                 type: "metricPathName"
   
               - name: "EVENT_ID"
                 type: "metricPathName"
   
               - name: "EVENT_CODE"
                 type: "metricValue"
   
               - name: "EVENT_POSTED_COUNT"
                 type: "metricValue"
   
           - displayName: "IO Usage"
             queryStmt: "Select NODE_NAME, READ_KBYTES_PER_SEC, WRITTEN_KBYTES_PER_SEC from IO_USAGE"
             columns:
               - name: "NODE_NAME"
                 type: "metricPathName"
   
               - name: "READ_KBYTES_PER_SEC"
                 type: "metricValue"
   
               - name: "WRITTEN_KBYTES_PER_SEC"
                 type: "metricValue"
   
           - displayName: "Node Status"
             queryStmt: "Select NODE_NAME, NODE_STATE from NODE_STATES"
             columns:
               - name: "NODE_NAME"
                 type: "metricPathName"
   
               - name: "NODE_STATE"
                 type: "metricValue"
                 properties:
                   convert:
                     "INITIALIZING" : 0
                     "UP" : 1
                     "DOWN" : 2
                     "READY" : 3
                     "UNSAFE" : 4
                     "SHUTDOWN" : 5
                     "RECOVERING" : 6
   
   
   numberOfThreads: 5
   

   ```



### Explanation of the type of queries that are supported with this extension ###
Only queries that start with **SELECT** are allowed! Your query should only return one row at a time. 
It is suggested that you only  return one row at a time because if it returns a full table with enormous amount of data, it may overwhelm the system and it may take a very long time to fetch that data.  
The extension does support getting values from multiple columns at once but it can only pull the metrics from the latest value from the row returned.

The name of the metric displayed on the **Metric Browser** will be the "name" value that you specify in the config.yml for that metric. 
Looking at the following sample query : 

```
 queries:
   - displayName: "Active Events"
     queryStmt: "Select NODE_NAME, EVENT_CODE, EVENT_ID, EVENT_POSTED_COUNT from Active_events"
     columns:
       - name: "NODE_NAME"
         type: "metricPathName"

       - name: "EVENT_ID"
         type: "metricPathName"

       - name: "EVENT_CODE"
         type: "metricValue"

       - name: "EVENT_POSTED_COUNT"
         type: "metricValue"

```

1. **queries** : You can add multiple queries under this field. 
    1. **displayName** : The name you would like to give to the metrics produced by this query. 
    2. **queryStmt** : This will be your SQL Query that will be used to query the database.
    3. **columns**: Under this field you will have to list all the columns that you are trying to get values from.
        1. **name** : The name of the column you would like to see on the metric browser.
        2. **type** : This value will define if the value returned from the column will be used for the name of the metric or if it is going to be the value of the metric.
            1. **metricPathName** : If you select this, this value will be added to the metric path for the metric.
            2. **metricValue** : If you select this, then the value returned will become your metric value that will correspond to the name you specified above.
            
For the query listed above, there will be two metrics returned as we have two columns of type "metricValue".
The metric path for them will be : 
1. Custom Metrics|SQL|Instance1|Active Events|NODE_NAME|EVENT_ID|EVENT_CODE
2. Custom Metrics|SQL|Instance1|Active Events|NODE_NAME|EVENT_ID|EVENT_POSTED_COUNT

## Credentials Encryption ##

Please visit [this page ](https://community.appdynamics.com/t5/Knowledge-Base/How-to-use-Password-Encryption-with-Extensions/ta-p/29397)to get detailed instructions on password encryption. The steps in this document will guide you through the whole process.

To set an encrypted password in config.yml, follow the steps below:

1. Download the util jar to encrypt the Credentials from [here](https://github.com/Appdynamics/maven-repo/blob/master/releases/com/appdynamics/appd-exts-commons/1.1.2/appd-exts-commons-1.1.2.jar).
2. Run command:

   	```
   	java -cp appd-exts-commons-1.1.2.jar com.appdynamics.extensions.crypto.Encryptor EncryptionKey CredentialToEncrypt

   	For example:
   	java -cp "appd-exts-commons-1.1.2.jar" com.appdynamics.extensions.crypto.Encryptor test password

     ```

3. Set the encryptionKey field in config.yml with the encryption key used, as well as the resulting encrypted password in encryptedPassword fields.


## Troubleshooting ##

Please follow the steps listed in this [troubleshooting-document] in order to troubleshoot your issue. 
These are a set of common issues that customers might have faced during the installation of the extension. 
If these don't solve your issue, please follow the last step on the [troubleshooting-document] to contact the support team.

## Contributing ##

Always feel free to fork and contribute any changes directly via [GitHub](https://github.com/Appdynamics/SQLMonitor).

## Community ##

Find out more in the [AppDynamics Exchange].

## Support ##

For any questions or feature request, please contact [AppDynamics Center of Excellence].

| Param | Description |
| ----- | ----- |
| Version |1.4   |
| Controller Compatibility |  3.7 or later  |
| SQL Version Tested On | SQL 4.1,4.2, MySQL |

[GitHub]: https://github.com/Appdynamics/SQLMonitor
[AppDynamics Exchange]: https://www.appdynamics.com/community/exchange/extension/sqlmonitor/
[AppDynamics Center of Excellence]: mailto:help@appdynamics.com
[troubleshooting-document]: https://community.appdynamics.com/t5/Knowledge-Base/How-to-troubleshoot-missing-custom-metrics-or-extensions-metrics/ta-p/28695