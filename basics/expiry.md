---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-05"

keywords: expiry, glacier, tier, s3, compatibility, api

subcollection: cloud-object-storage

---
{:external: target="blank" .external}
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:download: .download}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'} 
{:python: .ph data-hd-programlang='python'} 
{:http: .ph data-hd-programlang='http'} 

# Delete stale data with expiration rules
{: #expiry}

An expiration rule deletes objects after a defined period (from the object creation date). 

You can set the lifecycle for objects by using the web console, REST API, and third-party tools that are integrated with {{site.data.keyword.cos_full_notm}}. 

* An expiration rule can be added to a new or existing bucket.
* An existing expiration rule can be modified or disabled.
* A newly added or modified Expiration rule applies to all new and existing objects in the bucket.
* Adding or modifying lifecycle policies requires the `Writer` role. 
* Up to 1000 lifecycle rules (archive + expiration) can be defined per bucket.
* Allow up to 24 hours for any changes in Expiration rules to take effect.
* The scope of each expiration rule can be limited by defining an optional prefix filter to apply to only a subset of objects with names that match the prefix.
* An expiration rule without a prefix filter will apply to all objects in the bucket.
* The expiration period for an object, specified in number(s) of days, is calculated from the time the object was created, and is rounded off to the next day's midnight UTC. For example, if you have an expiration rule for a bucket to expire a set of objects ten days after the creation date, an object that was created on 15 April 2019 05:10 UTC will expire on 26 April 2019 00:00 UTC. 
* The expiration rules for each bucket are evaluated once every 24 hours. Any object that qualifies for expiration (based on the objects' expiration date) will be queued for deletion. The deletion of expired objects begins the following day and will typically take less than 24 hours. You will not be billed for any associated storage for objects once they are deleted.

## Attributes of Expiration Rules
{: #expiry-rules-attributes}

Each expiration rule has the following attributes:

* **ID**: Rule name
* **Expiration**: Time period for deletion of objects from the object creation date
* **Prefix**: An optional string that will be matched to the prefix of the object name in the bucket. A rule with a prefix will only apply to the objects that match.
* **Status**: A rule can either be enabled or disabled. A rule is active only when enabled. 

## Using the console
{: #expiry-using-console}

When creating a new bucket, check the **Add expiration rule** box. Next, click **Add rule** to create the new expiration rule. You can add up to five rules during bucket creation, and extra rules can be added later.

For an existing bucket, select **Configuration** from the navigation menu and click **Add rule** under the _Expiration rule_ section.

## Using the API and SDKs
{: #expiry-using-api-sdks}

You can programmatically manage expiration rules by using the REST API or the IBM COS SDKs. Select the format for the examples by selecting a category in the context switcher.

### Add an expiration rule to a bucket’s lifecycle configuration
{: #expiry-api-create}

**REST API reference**
{: http}

This implementation of the `PUT` operation uses the `lifecycle` query parameter to set lifecycle settings for the bucket. This operation allows for a single lifecycle policy definition for a bucket. The policy is defined as a set of rules consisting of the following parameters: `ID`, `Status`, `Filter`, and `Expiration`.
{: http}
 
Cloud IAM users must have the `Writer` role to remove a lifecycle policy from a bucket.

Classic Infrastructure Users must have `Owner` permissions on the bucket to remove a lifecycle policy from a bucket.

Header                    | Type   | Description
--------------------------|--------|----------------------------------------------------------------------------------------------------------------------
`Content-MD5` | String | **Required**: The base64 encoded 128-bit MD5 hash of the payload, which is used as an integrity check to ensure that the payload wasn't altered in transit.
{: http}

The body of the request must contain an XML block with the following schema:
{: http}

| Element                  | Type                 | Children                               | Ancestor                 | Constraint                                                                                 |
|--------------------------|----------------------|----------------------------------------|--------------------------|--------------------------------------------------------------------------------------------|
| `LifecycleConfiguration` | Container            | `Rule`                                 | None                     | Limit 1.                                                                                  |
| `Rule`                   | Container            | `ID`, `Status`, `Filter`, `Expiration` | `LifecycleConfiguration` | Limit 1000.                                                                                  |
| `ID`                     | String               | None                                   | `Rule`                   | Must consist of (`a-z,`A-Z0-9`) and the following symbols: `!` `_` `.` `*` `'` `(` `)` `-` |
| `Filter`                 | String               | `Prefix`                               | `Rule`                   | Must contain a `Prefix` element                                                            |
| `Prefix`                 | String               | None                                   | `Filter`                 | The rule applies to any objects with keys that match this prefix.                                                           |
| `Expiration`             | `Container`          | `Days` or `Date`                       | `Rule`                   | Limit 1.                                                                                  |
| `Days`                   | Non-negative integer | None                                   | `Expiration`             | Must be a value greater than 0.                                                           |
| `Date`                   | Date                 | None                                   | `Expiration`             | Must be in ISO 8601 Format and the date must be in the future.                            |
{: http}

The body of the request must contain an XML block with the schema that is addressed in the table (see Example 1).
{: http}

```xml
<LifecycleConfiguration>
	<Rule>
		<ID>id1</ID>
		<Filter />
		<Status>Enabled</Status>
		<Expiration>
			<Days>60</Days>
		</Expiration>
	</Rule>
</LifecycleConfiguration>
```
{: codeblock}
{: caption="Example 1. XML sample from the body of the request." caption-side="bottom"}
{: http}

**Syntax**
{: http}

```yaml
PUT https://{endpoint}/{bucket}?lifecycle # path style
PUT https://{bucket}.{endpoint}?lifecycle # virtual host style
```
{: caption="Example 2. Note the use of slashes and dots in this example of syntax." caption-side="bottom"}
{: codeblock}
{: http}

**Example request**
{: http}

```yaml
PUT /images?lifecycle HTTP/1.1
Host: s3.us.cloud-object-storage.appdomain.cloud
Date: Wed, 7 Feb 2018 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain
Content-MD5: M625BaNwd/OytcM7O5gIaQ==
Content-Length: 305

<LifecycleConfiguration>
	<Rule>
		<ID>id1</ID>
		<Filter />
		<Status>Enabled</Status>
		<Expiration>
			<Days>60</Days>
		</Expiration>
	</Rule>
</LifecycleConfiguration>
```
{: codeblock}
{: caption="Example 3. Request header samples for creating an object lifecycle configuration." caption-side="bottom"}
{: http}

**Code sample for use with NodeJS COS SDK**
{: javascript}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 
{: javascript}

```js
var aws = require('ibm-cos-sdk');
var ep = new aws.Endpoint('s3.us-south.cloud-object-storage.appdomain.cloud');
var config = {
    endpoint: ep,
    apiKeyId: 'ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE',
    ibmAuthEndpoint: 'https://iam.cloud.ibm.com/identity/token',
    serviceInstanceId: 'crn:v1:bluemix:public:cloud-object-storage:global:a/<CREDENTIAL_ID_AS_GENERATED>:<SERVICE_ID_AS_GENERATED>::',
};
var s3 = new aws.S3(config);
var date = new Date('June 16, 2019 00:00:00');

var params = {
  Bucket: 'STRING_VALUE', /* required */
  LifecycleConfiguration: {
    Rules: [ /* required */
      {
        Status: 'Enabled', /* required */
        ID: 'OPTIONAL_STRING_VALUE',
        Filter: {}, /* required */
        Expiration:
        {
          Date: date
        }
      },
    ]
  }
};

s3.putBucketLifecycleConfiguration(params, function(err, data) {
  if (err) console.log(err, err.stack); // an error occurred
  else     console.log(data);           // successful response
});
```
{: codeblock}
{: javascript}

**Code sample for use with Python COS SDK**
{: python}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 
{: python}

```python
import sys
import ibm_boto3
from ibm_botocore.client import Config

api_key = "ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE"
service_instance_id = "85SAMPLE-eDOb-4NOT-bUSE-86nnnb31eaxx"
auth_endpoint = "https://iam.cloud.ibm.com/identity/token"
service_endpoint = "https://s3.us-south.cloud-object-storage.appdomain.cloud"

cos = ibm_boto3.resource('s3',
                         ibm_api_key_id=api_key,
                         ibm_service_instance_id=service_instance_id,
                         ibm_auth_endpoint=auth_endpoint,
                         config=Config(signature_version='oauth'),
                         endpoint_url=service_endpoint)

response = cos.Bucket('<name-of-bucket>').put_bucket_lifecycle_configuration(
    Bucket='string',
    LifecycleConfiguration={
        'Rules': [
            {
                'Status': 'Enabled',
                'Filter': {},
                'Expiration':
                {
                    'Days': 123
                },
            },
        ]
    }
)

print("Bucket lifecyle: {0}".format(response))
```
{: codeblock}
{: python}

**Code sample for use with Java COS SDK**
{: java}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 
{: java}

```java
package com.ibm.cloud;

    import java.sql.Timestamp;
    import java.util.List;
    import java.util.Arrays;

    import com.ibm.cloud.objectstorage.ClientConfiguration;
    import com.ibm.cloud.objectstorage.SDKGlobalConfiguration;
    import com.ibm.cloud.objectstorage.auth.AWSCredentials;
    import com.ibm.cloud.objectstorage.auth.AWSStaticCredentialsProvider;
    import com.ibm.cloud.objectstorage.client.builder.AwsClientBuilder.EndpointConfiguration;
    import com.ibm.cloud.objectstorage.services.s3.AmazonS3;
    import com.ibm.cloud.objectstorage.services.s3.AmazonS3ClientBuilder;
    import com.ibm.cloud.objectstorage.services.s3.model.Bucket;
    import com.ibm.cloud.objectstorage.services.s3.model.BucketLifecycleConfiguration;
    import com.ibm.cloud.objectstorage.services.s3.model.ListObjectsRequest;
    import com.ibm.cloud.objectstorage.services.s3.model.ObjectListing;
    import com.ibm.cloud.objectstorage.services.s3.model.S3ObjectSummary;
    import com.ibm.cloud.objectstorage.oauth.BasicIBMOAuthCredentials;

    public class App
    {
        private static AmazonS3 _cosClient;

        /**
         * @param args
         */
        public static void main(String[] args)
        {
            SDKGlobalConfiguration.IAM_ENDPOINT = "https://iam.cloud.ibm.com/identity/token";
            String bucketName = "<sample-bucket-name>";
            String api_key = "ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE";
            String service_instance_id = "85SAMPLE-eDOb-4NOT-bUSE-86nnnb31eaxx"
            String endpoint_url = "https://s3.us-south.cloud-object-storage.appdomain.cloud";
            
            String storageClass = "us-south";
            String location = "us"; 
 
            _cosClient = createClient(api_key, service_instance_id, endpoint_url, location);
            
            // Define a rule for exiring items in a bucket
            int days_to_delete = 10;
            BucketLifecycleConfiguration.Rule rule = new BucketLifecycleConfiguration.Rule()
                    .withId("Delete rule")
                    .withExpirationInDays(days_to_delete)
                    .withStatus(BucketLifecycleConfiguration.ENABLED);
            
            // Add the rule to a new BucketLifecycleConfiguration.
            BucketLifecycleConfiguration configuration = new BucketLifecycleConfiguration()
                    .withRules(Arrays.asList(rule));
            
            // Use the client to set the LifecycleConfiguration on the bucket.
            _cosClient.setBucketLifecycleConfiguration(bucketName, configuration);   
        }
        
        /**
         * @param bucketName
         * @param clientNum
         * @param api_key
         * @param service_instance_id
         * @param endpoint_url
         * @param location
         * @return AmazonS3
         */
        public static AmazonS3 createClient(String api_key, String service_instance_id, String endpoint_url, String location)
        {
            AWSCredentials credentials;
            credentials = new BasicIBMOAuthCredentials(api_key, service_instance_id);

            ClientConfiguration clientConfig = new ClientConfiguration().withRequestTimeout(5000);
            clientConfig.setUseTcpKeepAlive(true);

            AmazonS3 cosClient = AmazonS3ClientBuilder.standard().withCredentials(new AWSStaticCredentialsProvider(credentials))
                    .withEndpointConfiguration(new EndpointConfiguration(endpoint_url, location)).withPathStyleAccessEnabled(true)
                    .withClientConfiguration(clientConfig).build();
            return cosClient;
        }
    }
```
{: codeblock}
{: java}
{: caption="Example 1. Code samples showing creation of lifecycle configuration." caption-side="bottom"}

### Examine a bucket’s lifecycle configuration, including expiration
{: #expiry-api-view}

This implementation of the `GET` operation uses the `lifecycle` query parameter to examine lifecycle settings for the bucket. An HTTP `404` response will be returned if no lifecycle configuration is present.
{: http}

Cloud IAM users must have the `Reader` role to remove a lifecycle policy from a bucket.

Classic Infrastructure Users must have `Read` permissions on the bucket to remove a lifecycle policy from a bucket.

Header                    | Type   | Description
--------------------------|--------|----------------------------------------------------------------------------------------------------------------------
`Content-MD5` | String | **Required**: The base64 encoded 128-bit MD5 hash of the payload, which is used as an integrity check to ensure that the payload wasn't altered in transit.
{: http}

**Syntax**
{: http}

```yaml
GET https://{endpoint}/{bucket}?lifecycle # path style
GET https://{bucket}.{endpoint}?lifecycle # virtual host style
```
{: caption="Example 5. Note the use of slashes and dots in this example of syntax." caption-side="bottom"}
{: codeblock}
{: http}

**Example Header Request**
{: http}

```yaml
GET /images?lifecycle HTTP/1.1
Host: s3.us.cloud-object-storage.appdomain.cloud
Date: Wed, 7 Feb 2018 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain
Content-MD5: M625BaNwd/OytcM7O5gIaQ==
Content-Length: 305
```
{: codeblock}
{: caption="Example 6. Request header samples for creating an object lifecycle configuration." caption-side="bottom"}
{: http}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration.
{: javascript}

```js
var aws = require('ibm-cos-sdk');
var ep = new aws.Endpoint('s3.us-south.cloud-object-storage.appdomain.cloud');
var config = {
    endpoint: ep,
    apiKeyId: 'ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE',
    ibmAuthEndpoint: 'https://iam.cloud.ibm.com/identity/token',
    serviceInstanceId: 'crn:v1:bluemix:public:cloud-object-storage:global:a/<CREDENTIAL_ID_AS_GENERATED>:<SERVICE_ID_AS_GENERATED>::',
};
var s3 = new aws.S3(config);

var params = {
  Bucket: 'STRING_VALUE' /* required */
};

s3.getBucketLifecycleConfiguration(params, function(err, data) {
  if (err) console.log(err, err.stack); // an error occurred
  else     console.log(data);           // successful response
});
```
{: codeblock}
{: javascript}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration.

```python
import sys
import ibm_boto3
from ibm_botocore.client import Config

api_key = "ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE"
service_instance_id = "85SAMPLE-eDOb-4NOT-bUSE-86nnnb31eaxx"
auth_endpoint = "https://iam.cloud.ibm.com/identity/token"
service_endpoint = "https://s3.us-south.cloud-object-storage.appdomain.cloud"

cos = ibm_boto3.resource('s3',
                         ibm_api_key_id=api_key,
                         ibm_service_instance_id=service_instance_id,
                         ibm_auth_endpoint=auth_endpoint,
                         config=Config(signature_version='oauth'),
                         endpoint_url=service_endpoint)

response = cos.Bucket('<name-of-bucket>').get_bucket_lifecycle_configuration(
    Bucket='string'
)

print("Bucket lifecyle: {0}".format(response))
```
{: codeblock}
{: python}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 

```java
package com.ibm.cloud;

    import java.sql.Timestamp;
    import java.util.List;
    import java.util.Arrays;

    import com.ibm.cloud.objectstorage.ClientConfiguration;
    import com.ibm.cloud.objectstorage.SDKGlobalConfiguration;
    import com.ibm.cloud.objectstorage.auth.AWSCredentials;
    import com.ibm.cloud.objectstorage.auth.AWSStaticCredentialsProvider;
    import com.ibm.cloud.objectstorage.client.builder.AwsClientBuilder.EndpointConfiguration;
    import com.ibm.cloud.objectstorage.services.s3.AmazonS3;
    import com.ibm.cloud.objectstorage.services.s3.AmazonS3ClientBuilder;
    import com.ibm.cloud.objectstorage.services.s3.model.Bucket;
    import com.ibm.cloud.objectstorage.services.s3.model.BucketLifecycleConfiguration;
    import com.ibm.cloud.objectstorage.services.s3.model.ListObjectsRequest;
    import com.ibm.cloud.objectstorage.services.s3.model.ObjectListing;
    import com.ibm.cloud.objectstorage.services.s3.model.S3ObjectSummary;
    import com.ibm.cloud.objectstorage.oauth.BasicIBMOAuthCredentials;
    
    public class App
    {
        private static AmazonS3 _cosClient;

        /**
         * @param args
         */
        public static void main(String[] args)
        {
            SDKGlobalConfiguration.IAM_ENDPOINT = "https://iam.cloud.ibm.com/identity/token";
            String bucketName = "<sample-bucket-name>";
            String api_key = "ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE";
            String service_instance_id = "85SAMPLE-eDOb-4NOT-bUSE-86nnnb31eaxx"
            String endpoint_url = "https://s3.us-south.cloud-object-storage.appdomain.cloud";
            
            String storageClass = "us-south";
            String location = "us"; 
 
            _cosClient = createClient(api_key, service_instance_id, endpoint_url, location);
            
            // Use the client to read the configuration
            BucketLifecycleConfiguration config = _cosClient.getBucketLifecycleConfiguration(bucketName);
            
            System.out.println(config.toString());
        }
        
        /**
         * @param bucketName
         * @param clientNum
         * @param api_key
         * @param service_instance_id
         * @param endpoint_url
         * @param location
         * @return AmazonS3
         */
        public static AmazonS3 createClient(String api_key, String service_instance_id, String endpoint_url, String location)
        {
            AWSCredentials credentials;
            credentials = new BasicIBMOAuthCredentials(api_key, service_instance_id);

            ClientConfiguration clientConfig = new ClientConfiguration().withRequestTimeout(5000);
            clientConfig.setUseTcpKeepAlive(true);

            AmazonS3 cosClient = AmazonS3ClientBuilder.standard().withCredentials(new AWSStaticCredentialsProvider(credentials))
                    .withEndpointConfiguration(new EndpointConfiguration(endpoint_url, location)).withPathStyleAccessEnabled(true)
                    .withClientConfiguration(clientConfig).build();
            return cosClient;
        }
        
    }
```
{: codeblock}
{: java}
{: caption="Example 2. Code samples showing inspection of lifecycle configuration." caption-side="bottom"}

### Delete a bucket’s lifecycle configuration, including expiration
{: #expiry-api-delete}

This implementation of the `DELETE` operation uses the `lifecycle` query parameter to examine lifecycle settings for the bucket. All lifecycle rules associated with the bucket will be deleted.  Transitions defined by the rules will no longer take place for new objects.  However, existing transition rules will be maintained for objects that were already written to the bucket before the rules were deleted.  Expiration Rules will no longer exist. An HTTP `404` response will be returned if no lifecycle configuration is present.
{: http}

Cloud IAM users must have the `Writer` role to remove a lifecycle policy from a bucket.

Classic Infrastructure Users must have `Owner` permissions on the bucket to remove a lifecycle policy from a bucket.

**Syntax**
{: http}

```yaml
DELETE https://{endpoint}/{bucket}?lifecycle # path style
DELETE https://{bucket}.{endpoint}?lifecycle # virtual host style
```
{: caption="Example 7. Note the use of slashes and dots in this example of syntax." caption-side="bottom"}
{: codeblock}
{: http}

**Example Header Request**
{: http}

```yaml
DELETE /images?lifecycle HTTP/1.1
Host: s3.us.cloud-object-storage.appdomain.cloud
Date: Wed, 7 Feb 2018 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain
Content-Length: 305
```
{: codeblock}
{: caption="Example 8. Request header samples for creating an object lifecycle configuration." caption-side="bottom"}
{: http}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 
{: javascript}

```js
var aws = require('ibm-cos-sdk');
var ep = new aws.Endpoint('s3.us-south.cloud-object-storage.appdomain.cloud');
var config = {
    endpoint: ep,
    apiKeyId: 'ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE',
    ibmAuthEndpoint: 'https://iam.cloud.ibm.com/identity/token',
    serviceInstanceId: 'crn:v1:bluemix:public:cloud-object-storage:global:a/<CREDENTIAL_ID_AS_GENERATED>:<SERVICE_ID_AS_GENERATED>::',
};
var s3 = new aws.S3(config);

var params = {
  Bucket: 'STRING_VALUE' /* required */
};

s3.deleteBucketLifecycleConfiguration(params, function(err, data) {
  if (err) console.log(err, err.stack); // an error occurred
  else     console.log(data);           // successful response
});
```
{: codeblock}
{: javascript}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 

```python
import sys
import ibm_boto3
from ibm_botocore.client import Config

api_key = "ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE"
service_instance_id = "85SAMPLE-eDOb-4NOT-bUSE-86nnnb31eaxx"
auth_endpoint = "https://iam.cloud.ibm.com/identity/token"
service_endpoint = "https://s3.us-south.cloud-object-storage.appdomain.cloud"

cos = ibm_boto3.resource('s3',
                         ibm_api_key_id=api_key,
                         ibm_service_instance_id=service_instance_id,
                         ibm_auth_endpoint=auth_endpoint,
                         config=Config(signature_version='oauth'),
                         endpoint_url=service_endpoint)

response = cos.Bucket('<name-of-bucket>').delete_bucket_lifecycle_configuration(
    Bucket='string'
)

print("Bucket lifecyle: {0}".format(response))
```
{: codeblock}
{: python}

Using the {{site.data.keyword.cos_full}} SDKs only requires calling the appropriate functions with the correct parameters and proper configuration. 
{: java}

```java
package com.ibm.cloud;

    import java.sql.Timestamp;
    import java.util.List;
    import java.util.Arrays;

    import com.ibm.cloud.objectstorage.ClientConfiguration;
    import com.ibm.cloud.objectstorage.SDKGlobalConfiguration;
    import com.ibm.cloud.objectstorage.auth.AWSCredentials;
    import com.ibm.cloud.objectstorage.auth.AWSStaticCredentialsProvider;
    import com.ibm.cloud.objectstorage.client.builder.AwsClientBuilder.EndpointConfiguration;
    import com.ibm.cloud.objectstorage.services.s3.AmazonS3;
    import com.ibm.cloud.objectstorage.services.s3.AmazonS3ClientBuilder;
    import com.ibm.cloud.objectstorage.services.s3.model.Bucket;
    import com.ibm.cloud.objectstorage.services.s3.model.BucketLifecycleConfiguration;
    import com.ibm.cloud.objectstorage.services.s3.model.ListObjectsRequest;
    import com.ibm.cloud.objectstorage.services.s3.model.ObjectListing;
    import com.ibm.cloud.objectstorage.services.s3.model.S3ObjectSummary;
    import com.ibm.cloud.objectstorage.oauth.BasicIBMOAuthCredentials;
    
    public class App
    {
        private static AmazonS3 _cosClient;

        /**
         * @param args
         */
        public static void main(String[] args)
        {
            SDKGlobalConfiguration.IAM_ENDPOINT = "https://iam.cloud.ibm.com/identity/token";
            String bucketName = "<sample-bucket-name>";
            String api_key = "ZRZDoNoUseOLL7bRO8SAMPLEHPUzUL_-fsampleyYE";
            String service_instance_id = "85SAMPLE-eDOb-4NOT-bUSE-86nnnb31eaxx"
            String endpoint_url = "https://s3.us-south.cloud-object-storage.appdomain.cloud";
            
            String storageClass = "us-south";
            String location = "us"; 
 
            _cosClient = createClient(api_key, service_instance_id, endpoint_url, location);
            
            // Delete the configuration.
            _cosClient.deleteBucketLifecycleConfiguration(bucketName);
            
            // Verify that the configuration has been deleted by attempting to retrieve it.
            config = _cosClient.getBucketLifecycleConfiguration(bucketName);
            String s = (config == null) ? "Configuration has been deleted." : "Configuration still exists.";
            System.out.println(s);
        }
        
        /**
         * @param bucketName
         * @param clientNum
         * @param api_key
         * @param service_instance_id
         * @param endpoint_url
         * @param location
         * @return AmazonS3
         */
        public static AmazonS3 createClient(String api_key, String service_instance_id, String endpoint_url, String location)
        {
            AWSCredentials credentials;
            credentials = new BasicIBMOAuthCredentials(api_key, service_instance_id);

            ClientConfiguration clientConfig = new ClientConfiguration().withRequestTimeout(5000);
            clientConfig.setUseTcpKeepAlive(true);

            AmazonS3 cosClient = AmazonS3ClientBuilder.standard().withCredentials(new AWSStaticCredentialsProvider(credentials))
                    .withEndpointConfiguration(new EndpointConfiguration(endpoint_url, location)).withPathStyleAccessEnabled(true)
                    .withClientConfiguration(clientConfig).build();
            return cosClient;
        }
        
    }

```
{: codeblock}
{: java}
{: caption="Example 3. Code samples showing deletion of lifecycle configuration." caption-side="bottom"}

## Next Steps
{: #expiry-next-steps}

Expiration is just one of many lifecycle concepts available for {{site.data.keyword.cos_full_notm}}.
Each of the concepts we've covered in this overview can be explored further at the 
[{{site.data.keyword.cloud_notm}} Platform](https://cloud.ibm.com/).

