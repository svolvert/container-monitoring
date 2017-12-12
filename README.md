# Logging and Monitoring for ECM Product Container

[Overview](#overview-of-logging-and-monitoring-for-ecm-product-container)   
[Requirements](#requirement)  
[Supported logging and monitoring services](#supported-logging-and-monitoring-services)  
[Prerequisites](#prerequisites)  
[Quickstart](#quickstart)  
[Customization](#customization)   
[Support](#support)  


## Overview

ECM containers provide logging and monitoring capabilities for troubleshooting, performance monitoring, and capacity planning based on monitoring data in real production environments. To simplify the enablement of these capabilities, the logging and monitoring components are built into each ECM product image. This means that you can start using monitoring and logging capabilities very easily by providing a few required environment variables and volumes. 

With the logging capability, you can ingest the system log, the Liberty log, and product logs into the back-end logging service you have chosen.

With the monitoring capability, you can collect system metrics like CPU, memory, IO, and network data related to ECM products through the PCH monitor plugin, and Liberty runtime information through the JMX plugin.

## Requirements

The following access is required:
* Access to the IBM ECM Docker store to pull the required image
* A Bluemix account for the logging and monitoring service to use IBM Bluemix Logmet monitoring and logging service
* Alternatively, you can host a local ELK (Elasticsearch/Logstash/Kibana) stack for logging service and Graphite/Grafana/Carbon stack for monitoring service

## Supported logging and monitoring services

You can use one of the following options for logging and monitoring:
* IBM Bluemix logging and monitoring service
* Open source ELK stack for logging service, and Graphite/Grafana/Carbon for monitoring service

If you are plannng to use logging and monitoring services other than the ones mentioned above, contact IBM.

## Prerequisites

Use the following environment variables for both supported and any new external logging and monitoring services.

### Environment Variables

* Use with Bluemix logging and monitoring service:

|Name|Description|Required|Default Value|
|----|-----------|--------|-------------|
|MON\_METRICS\_WRITER\_OPTION|Use the setting below to determine the monitoring service: |Yes||
||2: IBM Cloud monitoring service |||
|MON\_METRICS\_SERVICE\_ENDPOINT|Bluemix metrics service endpoint using the format host:port|No|metrics.ng.bluemix.net:9095|
|MON\_BMX\_GROUP|Bluemix metrics group name ended with ".", which is used for metrics prefix.|No|com.ibm.ecm.monitor.|
|MON\_BMX\_SPACE\_ID|Bluemix space/tenant id| Yes, but only required if MON\_METRICS\_WRITER\_OPTION is 1 or 2|N/A|
|MON\_BMX\_API\_KEY|Bluemix API key, which is used for all the RESTful APIs|Yes, but only required if MON\_METRICS\_WRITER\_OPTION is 2. Refer to https://pages.github.ibm.com/metrics-service/api-reference/iam-authentication.html||
|MON\_BMX\_METRICS\_SCOPE\_ID|GUID that identifies the space or organization domain where the metrics are stored. See https://pages.github.ibm.com/metrics-service/collectd/collectd-monitoring_plugin.html for more detail.|Yes, but only required if MON\_METRICS\_WRITER\_OPTION is 2 and the scope is for organization or space, otherwise, it is not required for account scope. If the scope ID is set, it should be prefixed with o- for organization and s- for space. Refer to https://pages.github.ibm.com/metrics-service/api-reference/iam-authentication.html for how to determin the space or organization GUID.||
|MON\_LOG\_SHIPPER\_OPTION|Use one of the following settings to determine for the log shipper:|Yes|Set to 2 to use Logstash for Bluemix logging service|
||1: Filebeat|||
||2: Logstash mtlumberjack output for Bluemix logging service|||
|MON\_LOG\_SERVICE\_ENDPOINT|Logging service endpoint using format host1:port,host2:port,...,hostN:port . The port for all hosts must be same|No|logs.opvis.bluemix.net:9091|
|MON\_BMX\_LOGS\_LOGGING\_TOKEN|Bluemix logging service token|Yes||
|MON\_LOG\_PATH|The log file path to be collected. Wildcard is supported in each file path, and multiple paths can be delimitered by colon (:), and the whole path string should be surrounded with single/double quote if blank space is contained in path. The following paths is not required to be specified in this varible because they have already been included by default.|Yes||
|| - /var/log/messages |||
|| - /var/log/syslog|||
|| - /var/log/supervisor/\*.log|||
|| - /var/log/secure|||
|| - /var/log/collectd.log|||
|| - /var/log/PCHMonitor.log|||
|| - /var/log/MBeanMonitor.log|||
||The paths should never be duplicated for other logs. As an example, MON\_LOG\_PATH=/output/FileNet/\*/\*.log:/logs/\*/\*.log. Also, the placeholder {{HOSTNAME}} can be used to set dynamic path like MON_LOG_PATH=/output/FileNet/{{HOSTNAME}}/\*.log |||
|ECM\_PRODUCT\_TYPE|The ECM product type, such as CPE, ICM, ICN, CSS, CMIS, etc|Yes|
|MON\_PCH\_SERVICE\_ENDPOINT|PCH service endpoint with format host:port|No|localhost:32775|
|MON\_ECM\_METRICS\_COLLECT\_INTERVAL|The time interval for collecting metrics. An interval value less than 60 seconds is not supported by PCH|No|60 seconds|
|MON\_ECM\_METRICS\_FLUSH\_INTERVAL|The time interval for flushing metrics to the monitoring service|No|60 seconds|
|MON\_PCH\_LOG\_METRICS\_FLAG|Enable log metrics for PCH|No|true|
|MON\_PCH\_LOG\_STATS\_FLAG|Enable log stats for PCH|No|true|
|MON\_PCH\_LOG\_COUNTERS\_FLAG|Enable log counter for PCH|No|true|
|TZ|Timezone, see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones|No|Etc/UTC|


* Use with self-hosted logging and monitoring server:

|Name|Description|Required|Default Value|
|----|-----------|--------|-------------|
|MON\_METRICS\_WRITER\_OPTION|Use this setting to determine the monitoring server, which can be null or 0 when using self-hosted monitoring server  |No|default is null, which means to use self-hosted monitoring server|
|MON\_METRICS\_SERVICE\_ENDPOINT|Self-hosted metrics service, such as Graphite, with the endpoint using the format host:port|Yes|No|
|MON\_LOG\_SHIPPER\_OPTION|Use this setting to indicate which log shipper to use. The valid options are:|Yes||
||1: Filebeat|||
||2: Logstash mtlumberjack output for Bluemix logging service|||
|MON\_LOG\_SERVICE\_ENDPOINT|Logging service endpoint using the format host1:port,host2:port,...,hostN:port . The port for all hosts must be the same|Yes||
|MON\_LOG\_PATH|The log file path to be collected. Wildcard is supported in each file path, multiple paths can be delimited by a colon (:), and the whole path string should be surrounded with single/double quote if a blank space is contained in the path. The following paths are not required in this varible because they have already been included by default.|Yes||
|| - /var/log/messages |||
|| - /var/log/syslog|||
|| - /var/log/supervisor/\*.log|||
|| - /var/log/secure|||
|| - /var/log/collectd.log|||
|| - /var/log/PCHMonitor.log|||
|| - /var/log/MBeanMonitor.log|||
||The paths should never be duplicated for other logs. As an example, MON\_LOG\_PATH=/output/FileNet/\*/\*.log:/logs/\*/\*.log. Also, the placeholder {{HOSTNAME}} can be used to set a dynamic path like MON_LOG_PATH=/output/FileNet/{{HOSTNAME}}/\*.log |||
|ECM\_PRODUCT\_TYPE|The ECM product type, such as CPE, ICM, ICN, CSS, CMIS, etc|Yes|
|MON\_PCH\_SERVICE\_ENDPOINT|PCH service endpoint with format host:port|No|localhost:32775|
|MON\_ECM\_METRICS\_COLLECT\_INTERVAL|The time interval for collecting metrics, an interval value less than 60 seconds is not supported by PCH|No|60 seconds|
|MON\_ECM\_METRICS\_FLUSH\_INTERVAL|The time interval for flushing metrics to the backend monitoring service|No|60 seconds|
|MON\_PCH\_LOG\_METRICS\_FLAG|Enable log metrics for PCH|No|true|
|MON\_PCH\_LOG\_STATS\_FLAG|Enable log stats for PCH|No|true|
|MON\_PCH\_LOG\_COUNTERS\_FLAG|Enable log counter for PCH|No|true|
|TZ|Timezone, see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones|No|Etc/UTC|


[*] The variables in the previous two tables are required to activate monitoring and logging capability. The variables are not required if you do not want to activate monitoring and logging. 

### Volumes

If you want to use open-sourced ELK stack for your logging service and the Logstash server requires authentication by an SSL certificate, you mount the public certificate generated for/from the Logstash server into the volume **/etc/pki/tls/certs/**. The certificate should be named as **logstash.crt**. 

Request offical SSL certificate and a private key for the Logstash server in a production environment. You can use openssl to generate the required certificates. For example:

~~~ 
mkdir -p /var/lib/ecm/logmet/certs
mkdir -p /var/lib/ecm/logmet/private
cd /var/lib/ecm/logmet
openssl req -config /etc/ssl/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash.key -out certs/logstash.crt
~~~

|Container folder/file|Host directory example|Description|
|----------------|----------------------|-----------|
|/etc/pki/tls/certs|/var/lib/ecm/logmet/certs|SSL certificate for Logstash server|


To mount the requested SSL certificate, run the container by using a command like the following example:

~~~
docker run -v /var/lib/ecm/logmet/certs:/etc/pki/tls/certs:ro {... others}
~~~


## Quickstart

The ECM monitoring Docker image is bundled with an ECM product. You cannot run this image on its own. The following information provides a  quick start example for one of ECM products, the Content Platform Engine image.

### Download the ECM product Docker image.

~~~
docker login  -u [Docker ID] -p [Password]
docker pull ecmcontainers/ecm_earlyadopters_cpe:earlyadopters-gm5.5
~~~

### Enable the logging and monitoring service.

You can choose **any** one of the following methods.  

- Use IBM Bluemix logmet service with a Bluemix account: 

~~~
docker run -d -e MON_METRICS_WRITER_OPTION=2 -e MON_METRICS_SERVICE_ENDPOINT=metrics.ng.bluemix.net:9095 -e MON_BMX_GROUP=com.ibm.ecm.monitor. -e MON_BMX_METRICS_SCOPE_ID={space or organization guid} -e MON_BMX_API_KEY={IAM API key} -e MON_LOG_SHIPPER_OPTION=2 -e MON_BMX_SPACE_ID={tenant id} -e MON_LOG_SERVICE_ENDPOINT=logs.opvis.bluemix.net:9091 -e MON_BMX_LOGS_LOGGING_TOKEN={log logging token} --hostname=cpe-logmet-test -p 9080:9080 -p 9443:9443 --name cpe -e CONTAINERTYPE=1 -e CPESTATICPORT=false -v /tmp/data/asa:/opt/ibm/asa -v /tmp/data/textext:/opt/ibm/textext -v /tmp/data/icmrules:/opt/ibm/icmrules -v /tmp/data/logs:/opt/ibm/wlp/usr/servers/defaultServer/logs -v /tmp/data/FileNet:/opt/ibm/wlp/usr/servers/defaultServer/FileNet -v /tmp/data/configDropins/overrides:/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides -v /tmp/data/bootstrap:/opt/ibm/wlp/usr/servers/defaultServer/lib/bootstrap ecmcontainers/ecm_earlyadopters_cpe:earlyadopters-gm5.5
~~~


- Alternatively, use your own ELK server and Carbon/Graphite server:

This requires logstash.crt to configure for ELK server (Logstash). Place the certificate (logstash.crt) to the Docker host directory, such as /var/lib/ecm/logmet/certs/logstash.crt, and mount the directory /etc/pki/tls/certs/logstash.crt to a volume inside the container.

~~~
docker run -d -e MON_METRICS_SERVICE_ENDPOINT="{your carbon service endpoint}" -e MON_LOG_SHIPPER_OPTION=1 -e MON_LOG_SERVICE_ENDPOINT="{your Logstash service endpoint}" -v /tmp/certs/{your Logstash server certificate file}:/etc/pki/tls/certs/logstash.crt:ro --hostname=cpe-logmet-test -p 9080:9080 -p 9443:9443 --name cpe -e CONTAINERTYPE=1 -e CPESTATICPORT=false -v /tmp/data/asa:/opt/ibm/asa -v /tmp/data/textext:/opt/ibm/textext -v /tmp/data/icmrules:/opt/ibm/icmrules -v /tmp/data/logs:/opt/ibm/wlp/usr/servers/defaultServer/logs -v /tmp/data/FileNet:/opt/ibm/wlp/usr/servers/defaultServer/FileNet -v /tmp/data/configDropins/overrides:/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides -v /tmp/data/bootstrap:/opt/ibm/wlp/usr/servers/defaultServer/lib/bootstrap ecmcontainers/ecm_earlyadopters_cpe:earlyadopters-gm5.5
~~~

## Customization

In order to enable logging and monitoring for ECM product containers, the environment variables that are related to logging and monitoring must be passed as command line arguments. Note that the term endpoint below is usually in a format like **host:port**. 

Connect to the Logstash server and Carbon server where the ECM product container can send logs by using the specified log shipper FileBeat:  

    docker run -d -e MON_METRICS_SERVICE_ENDPOINT="{your carbon service endpoint}" -e MON_LOG_SHIPPER_OPTION=1 -e MON_LOG_SERVICE_ENDPOINT="{your Logstash service endpoint}" --hostname={container hostname} {arguments of product image} {product image name}    
        
Connect to the IBM Bluemix metrics service using IBM Cloud monitoring metrics for an account scope and Logmet logging service using multi-tenant lumberjack protocol: 

    docker run -d -e MON_METRICS_WRITER_OPTION=2 -e MON_METRICS_SERVICE_ENDPOINT=metrics.ng.bluemix.net:9095 -e MON_BMX_GROUP=com.ibm.ecm.monitor. -e MON_BMX_API_KEY={IAM API key} -e MON_LOG_SHIPPER_OPTION=2 -e MON_BMX_SPACE_ID={tenant id} -e MON_LOG_SERVICE_ENDPOINT=logs.opvis.bluemix.net:9091 -e MON_BMX_LOGS_LOGGING_TOKEN={log logging token} --hostname={container hostname} {arguments of product image} {product image name}  

You can access [IBM Bluemix Logmet Service Token](#ibm_bluemix_logmet_service_token) to know the generation of above token or API key.

Connect to Bluemix metrics service using IBM Cloud monitoring metrics for a space or organization scope and Logmet logging service using multi-tenant lumberjack protocol:  

    docker run -d -e MON_METRICS_WRITER_OPTION=2 -e MON_METRICS_SERVICE_ENDPOINT=metrics.ng.bluemix.net:9095 -e MON_BMX_GROUP=com.ibm.ecm.monitor. -e MON_BMX_METRICS_SCOPE_ID={space or organization guid} -e MON_BMX_API_KEY={IAM API key} -e MON_LOG_SHIPPER_OPTION=2 -e MON_BMX_SPACE_ID={tenant id} -e MON_LOG_SERVICE_ENDPOINT=logs.opvis.bluemix.net:9091 -e MON_BMX_LOGS_LOGGING_TOKEN={log logging token} --hostname={container hostname} {arguments of product image} {product image name}  
    
You can access [IBM Bluemix Logmet Service Token](#ibm_bluemix_logmet_service_token) to know the generation of above token or API key.

## IBM Bluemix Logmet Service Token

To access IBM Cloud logging and monitoring backend service, you need follow the steps below to get authentication token or API key.

1. Create a IBM BMX account if one does not exist

   http://console.ng.bluemix.net

2. Create BMX organization & space

   https://console.bluemix.net/docs/admin/orgs_spaces.html#orgsspacesusers

3. Assign users to the BMX org/space

   https://console.bluemix.net/docs/iam/iamuserinv.html#iamuserinv

4. Generate an IAM token which will be used as part of generating authentication tokens for IBM Cloud Monitoring plugin and IBM Cloud Logging plugin

   https://console.bluemix.net/docs/services/cloud-monitoring/security/auth_iam.html#auth_iam

5. Generate authentication token for IBM Cloud logging

   https://pages.github.ibm.com/alchemy-logmet/getting-started/authentication.html


### Get authentication token for IBM Cloud logmet logging service plugin

1.	bx login -a api.ng.bluemix.net --sso  

2.	Get the temporary token to authenticate per prompt in step 1

3.  bx target -o <ORG>  
    Replace ORG with Organization name  
    
4.	bx iam space <SPACE_Name> --guid  
    This is to get space id for the space name which will be used for environment variable **MON_BMX_SPACE_ID**    
    
5.	bx iam oauth-tokens  

6.	Use the generated space id and auth token to generate the metrics token using command below and save it for later to use as part of configuring container to send metrics data to IBM Cloud Monitoring

    curl -k -XGET -H "X-Auth-Project-Id:SPACE_ID" -H "X-Auth-Token:OAUTH_TOKEN" https://logging.ng.bluemix.net/token

7. Use the generated logging token as value for the environment variable **MON_BMX_LOGS_LOGGING_TOKEN** when creating product container.
          

### Get IBM Cloud API Key & IAM Policy for IBM Cloud Monitoring collectd plugin.

Use below documentation link to create API key and IAM Policy for IBM Cloud Monitoring plugin.

https://pages.github.ibm.com/metrics-service/api-reference/iam-authentication.html

Use the generated API key as a value for the environment variable **MON_BMX_API_KEY** when creating product container.

Support can be obtained at [IBM® DeveloperWorks Answers](https://developer.ibm.com/answers/)
<br>
Use the ECM-CONTAINERS tag and assistance will be provided.<br>
*Note: Limited support available during Early Adopter Program*
