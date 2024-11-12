---
tags: [aws]
---
# Simple Storage Service (S3)

S3 is an AWS service that stores data as *objects*. Object is a tuple of:

- data -- the data to save (e.g. image, text file, audio)
- metadata -- permissions, how it is saved, dates, ...
- key -- identifier of the data

Maximum size of a single object's data is **5TB**. Objects are placed into
*buckets*.

## Characteristics

S3 does the basics:

- optional versioning
- redundancy built-in -- S3 stores single object in at least 3
  [AZs](./aws_infrastructure.md)
- most durable storage -- object will be intact after a year with
  99.999999999% probability

## Storage classes

The optimal storage back-end may differ for each object based on how it is used.
That's why Amazon offers several *Storage Classes*, which differ in pricing,
availability and redundancy. You can read more at [aws
docs](https://aws.amazon.com/s3/storage-classes/).


- *S3 Standard* -- high availability, frequent access
- *S3 Standard-Infrequent Access* (*S3 Standard-IA*) -- S3 Standard but
    - lower price
    - higher retrieval price
- *S3 One Zone-Infrequent Access* (*S3 One Zone-IA*) -- S3 Standard-IA but
    - even lower price
    - lower redundancy -- data stored in only **one** AZ
- *S3 Glacier Instant Retrieval*
    - low cost for storage
    - data available within **few milliseconds**
    - optimized for archival of data that should be immediately accessible (e.g.
      backups)
- *S3 Glacier Flexible Retrieval*
    - low cost for storage
    - data available within **few minutes to hours**
    - optimized for archival
- *S3 Glacier Deep Archive*
    - low cost for storage
    - data available within **12 hours**
    - optimized for archival

Objects can also move from one class to another. In other words object's data
class can be dynamic in time. There are two options:

- automatic -- *Intelligent-Tiering*
    - small monitoring fee per object
    - each object is assigned the optimal storage class based on past accesses
      to that object
- manual -- *S3 Lifecycle*
    - based on a predefined policy, objects move from one class to another

### S3 Outposts

S3 can be deployed in [AWS Outposts](./aws_infrastructure.md). If so, S3 uses
'outposts' storage class, optimized for single-region situation.
