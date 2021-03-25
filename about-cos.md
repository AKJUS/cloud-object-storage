---

copyright:
  years: 2017, 2021
lastupdated: "2021-03-25"

keywords: about, object storage, overview, erasure coding, multiple writes, availability zone, bucket, integrity

subcollection: cloud-object-storage


---
{:new_window: target="_blank"}
{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:faq: data-hd-content-type='faq'}
{:support: data-reuse='support'}

# What is {{site.data.keyword.cos_full_notm}}?
{: #about-cloud-object-storage}

{{site.data.keyword.cos_full}} is a highly available, durable, and secure platform for storing unstructured data.  Unstructured data (sometimes called binary or "blob" data) refers to data that is not highly structured in the manner of a database. Object storage is the most efficient way to store PDFs, media files, database backups, disk images, or even large structured datasets.
{: shortdesc}

The files that are uploaded into {{site.data.keyword.cos_full_notm}} are called **objects**.  Objects can be anywhere from very small (a few bytes) [to very large] (up to 10TB).  They are organized into **buckets** that serve as containers for objects, and which can be configured independently from one another in terms of locations, resiliency, billing rates, security, and object lifecycle. Objects themselves have their own metadata in the form of user-defined tags, legal holds, or archive status.  Within a bucket, the hierarchy of objects is effectively "flat", although it is possible to add prefixes to object names to provide some organization and to provide flexibility in listing and other operations.  

{{site.data.keyword.cos_full_notm}} is strongly consistent for all data operations, and eventually consistent for bucket configuration operations. This means that when an object is uploaded, the server responds with a `200 OK` after the object is successfully written, and the object is immediately available for listing and reading.  All data stored in {{site.data.keyword.cos_full_notm}} is encrypted, erasure-coded, and dispersed across three locations (with the distance between locations ranging from within a single data center, across a Multi-Zone Region or MZR, or even across multiple MZRs). This geographic range of dispersal contributes to a bucket's resiliency.

All requests and responses are made over HTTPS and all requests support the use of hash-based integrity checks using a `Content-MD5` header. If the provided MD5 hash does not match the checksum computed by the storage service, the object is discarded and an error is returned. All `GET` and `HEAD` requests made to objects return an `Etag` value with the MD5 hash of the object to ensure integrity on the client side. 

Developers use APIs to interact with their object storage. {{site.data.keyword.cos_full_notm}} supports a subset of the S3 API for reading and writing data, as well as for bucket configuration. Additionally, there is a {{site.data.keyword.cos_short}} Resource Configuration API for reading and configuring bucket metadata. Software development kits (SDKs) are available for the Python, Java, Go, and the Node.js framework. A plug-in is available for the [{{site.data.keyword.cloud_notm}} Command Line Interface](/docs/cli?topic=cli-getting-started). 

The [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/){: external} provides a user interface for most operations and configuration as well. 

Users of {{site.data.keyword.cos_full_notm}} refer to their binary data, such as files, images, media, archives, or even entire databases as objects. Objects are stored in a bucket, the container for their unstructured data. Buckets contain both inherent and user-defined metadata. Finally, objects are defined by a globally unique combination of the bucket name and the object key, or name.

## For users of Cloud Object Storage IaaS
{: #iaas}

IaaS users migrating to the IAM enabled service can reference this documentation, but not all features are supported, and naturally only HMAC authentication can be used.

## Next Steps
{: #about-cos-next-steps}

Documentation on the best way to [get started](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage) provides support to provision accounts, to create buckets, to upload objects, and to use a reference of common operations through API interactions.


