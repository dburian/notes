---
tags: [aws]
---
# Storage in AWS

This note documents the contents of Module 5 of [AWS Cloud Practitioner
Essentials](./sources/aws_cloud_practitioner.md).

There are quite a few services related to storing data. I divided the services
into 4 categories:

- storage services -- allow to save a piece of data
- database services -- more managed services with more restrictions on data,
  plus query features
- purpose-built services -- built for particular purpose only
- complementary services -- do not store data, but interact with the before
  mentioned services

## Storage services

- empheral storage -- storage that disappears after EC2 instance is
  terminated or stopped 
    - *Instance Stores*
        - virtual file systems stored on hard drives of current physical machine
          hosting the running EC2 instance
        - the file system is cleaned once EC2 is stopped or terminated
- block storage -- data is split to blocks which enables more precise read/write
  operations such as writing/reading only part of a file
    - *Elastic Block Store* (*EBS*)
        - provides volumes of specific types and size
        - enables backups using incremental *EBS snapshots*
        - stores data in single [AZ](./aws_infrastructure.md), this means
          that the EC2 instance and EBS must be running in the **same AZ zone**
        - does **not** automatically scale
- object storage -- stores objects in a filesystem-like structure
    - [*Simple Storage Service* (*S3*)](./aws_simple_storage_service.md)
- file system storage -- stores files in a virtual filesystem
    - *Elastic File System* (*EFS*)
        - allows **shared** access to files
        - uses block storage under the hood
        - grows and shrinks automatically
        - regional service -- stores data in **multiple AZs**
        - automatic horizontal scaling

## Database services

- relational database services -- stores relational tables queryable via SQL
    - *Relational Database Service* (*RDS*)
        - stores relational tables with configurable back-end e.g. MySQL, Oracle,
          Postgress, ...
        - managed -- automatically backs up data, does database setup and
          patching, and hardware provisioning
    - *Amazon Aurora*
        - whole managed relational database solution
        - uses S3 for faster access times while reducing I/O operations
        - for high availability scenarios -- 6 copies across 3 AZs
        - automatic backups to S3
- key-value database services -- stores items w/ attributes under an identifier,
  with query language that is simpler and less powerful than SQL
    - *DynamoDB*
        - automatically scalable
        - high availability
        - redundancy built-in -- copies across several AZs
- document databases -- stores documents
    - *DocumentDB*
        - stores documents using MongoDB
- graph databases -- stores graphs
    - *Amazon Neptune*
- immutable databases -- stores never-changing records
    - *Quantum Ledger Database* (*QLDB*)

## Purpose-built storage services

- *Redshift*
    - data warehouse service -- highly scalable solution for storing large
      streams of data
    - able to ingest many data streams
    - features to query and analyze data -- build for understanding data
      relationships (e.g. analytics)

- *Managed Block Chain*
    - create and manage block chains

## Complementary services

There are also services that do interact with storage/database services, while
not actually storing data:

- *AWS Database Migration Service* (*DMS*)
    - allows to move data from one database to another
    - the source and target databases do not have to be of the same type
    - enables keeping the source database functional at the time of migration
    - useful when:
        - migrating data (duh)
        - mirroring production data in a dev environment without endangering
          production
        - consolidating data from multiple databases

- *ElastiCache*
    - add cache layers to any database to improve read speeds
    - can use either Redis or memory

- *DynamoDB Accelerator*
    - in-memory cache for *DynamoDB*
