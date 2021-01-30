# MSSQL-ELK
Loading data from SQL Server to ELK stack

Download the SQL Server 2019 bits and the elk 7.10.2 suite for windows from the below links.

https://download.microsoft.com/download/d/a/2/da259851-b941-459d-989c-54a18a5d44dd/SQL2019-SSEI-Dev.exe

https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-windows-x86_64.zip

https://artifacts.elastic.co/downloads/logstash/logstash-7.10.2-windows-x86_64.zip

https://artifacts.elastic.co/downloads/kibana/kibana-7.10.2-windows-x86_64.zip

Post SQL Server 2019 developer edition installation restore the below AdventureWorks database.

https://github-production-release-asset-2e65be.s3.amazonaws.com/53698446/b14c8c28-bb2b-11e7-9547-07167e8769f0?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210128%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20210128T095242Z&X-Amz-Expires=300&X-Amz-Signature=93a66320b895b09d0c9c697e78688930bf37f197512a6a9227327031b2c5e44c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=53698446&response-content-disposition=attachment%3B%20filename%3DAdventureWorks2014.bak&response-content-type=application%2Foctet-stream

Verify that we have the JDK tools installed on the machine on which the installation is being carried out.

Install ELK on Windows by extracting it to the C drive.

C:\elasticsearch-7.10.2\elasticsearch-7.10.2>bin\elasticsearch
C:\logstash-7.10.2\logstash-7.10.2>bin\logstash
C:\kibana-7.10.2\kibana-7.10.2>bin\kibana

Post installtion we will be to check the services on the below links.
elasticsearch : http://localhost:9200/
kibana : http://localhost:5601/

Note :  to use the logstash to connect to SQL server we need the SQLJDBC driver to be installed, use the below link to download extract it and place it on the jars folder location C:\logstash-7.10.2\logstash-7.10.2\logstash-core\lib\jars

https://download.microsoft.com/download/F/0/F/F0FF3F95-D42A-46AF-B0F9-8887987A2C4B/sqljdbc_4.2.8112.200_enu.tar.gz

For the data movement from SQL Server AdventureWorks database to elasticsearch, we need to create a configuration file with the input filter and output parameter.
as specified the the sql.conf file. 

In the below sample we are selecting the complete adventureworks2014.sales.salesterritory table and loading the data into the adventureworks_salesterritory index which will have a single type salesterritory created.

# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
  jdbc {
    jdbc_connection_string => "jdbc:sqlserver://localhost:1433;databasename=adventureworks2014;user=sa;password=SQLDBA@2021"
    jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
    jdbc_driver_library => ["C:\logstash-7.10.2\logstash-7.10.2\logstash-core\lib\jars\sqljdbc42.jar"]
    jdbc_user => "sa"
    jdbc_password => "SQLDBA@2021"
    schedule => "* * * * *"
    statement => "SELECT * FROM adventureworks2014.sales.salesterritory"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "adventureworks_salesterritory"
  }
}

We are not applying any filters in our config we are just commanding logstash to push the sql query load to elasticsearch index and this to be run every minute as we have use schedule option with "* * * *".

Once we have the configuration file ready we have to place it in the bin folder of logstash and run the below command to start the load.

logstash -f sql.conf

We will be able to see the records being pushed as documents on the elasticsearch index.

This flow can be modified as per your need by modifying the logstash configuration file and later visualize using kibana.

