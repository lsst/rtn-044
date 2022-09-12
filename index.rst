:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========

Postgres database design for Butler in the USDF.  Includes details on Postgres architecture in Kubernetes, authentication, backups, and monitoring/logging. 


Architecture
============

Postgres is deployed on Kubernetes using the CloudNativePG (CNPG) Kuberneter opeator.  CNPG is an open source project developed by Enterprise DB.  The CNPG Operator includes built in capabilities to manage upgrades, high availability, replication, and backup.  Commerical support is available for a fee.

A seperate development Kubernetes cluster and production Kubernetes cluster are deployed.  This allows for testing both operator functionality and configuration prior to deploying in production.

The following are outstanding questions.

- A Butler registry will be distinct for each data release.  Will there be a seperate cluster per release or database?  Will joins be needed across data registries?

For scaling read replicas will be provisioned to handle the read load.  Standby replicas are exposed through a seperate PgBouncer read only instance.

Version
=======

Postgres version 14 is deployed. Previously Postgres 12 was deployed at NCSA and Postgres 13 at IDF.  Postgres 14 has performance improvments and is the latest stable major version.  There are no known limitations that would prevent Butler from running on Postgres 14.

Storage
=======

The storage for the CNGP clusters is on Weka storage using the Container Storage Interface (CSI) driver.  1,000 GB is provisioned for production.  Volume expansion is supported to increase the size of the disks.  Both CNGP and the Weka CSI Plugin support for volume expansion.

Total storage is forecast to be 100s of Terabytes per year.


Access Methods
==============

PgBouncer is a lightweight connection pooler for client connections.  PgBouncer front ends connections for connection pooling and protecting access to the database.  All end user access to the database will be through Butler.  Butler uses SQL Alchemy in driver mode and not ORM mode.  No direct access through PSQL or other Postgres administrative tools is required by end users.  Butler Postgres does not require external connectivty outside of USDF so no external IP address is needed on the database.

Postgres administrators will be able to use PSQL and other tools to perform administrative functions.  PSQL can be run through the S3DF Rubin Servers.


Authentication and Access Control
=================================

Security requirements for authentication and authorization are:

- Expected user count is 200?
- Limit management overhead as there is not staff to reset passwords
- Track activity by user to tell who made changes
- Is there a hybrid approach for read only users vs developers/admins?

There are four types of access needed to Butler Postgres.

#. Read only access - Read data through Butler
#. Developer write access - Write data to Butler
#. PaNDA Service account - Query butler from jobs and store results of job runs
#. Administrative access - Create databases, tables, edit roles

For individual user accounts in Postgres a username needs to be created and role assigned.  


- Shared Username (rubin) with shared password
   - Pros
      - There is no support needed to reset passwords
      - Easy to deploy and build into image without end user intervention
      - Remove risk of an developer creating a production database with only their ownership and schema
   - Cons:
      - Not able to track who made changes

- Individual username with shared password
   - Pros:
      - Able to track who made changes
   - Cons:
      - Additional support overhead
      - Security implications of shared password

- Individual username with passwords stored in Vault
   - Pros:
      - Able to track who made changes
   - Cons:
      - Additional support overhead to manage passwords and expirations

- LDAP
    - Cons
      - Does not currently work.  LDAPS and LDAP with Start TLS were tested for authentication. An unknown error was returned by Postgres.  It also appears that PG Bouncer does not support LDAP based on an open issue in the PG Bouncer GitHub repository.  The following options are available for authentication.

scram-sha-256 will be used for password encryption as is now is the default for Postgres 14.  This encryption method was previously used by Butler in other environments.


Backups
=======

CNPG has built in backups through Barman.  Backups are integrated with the WAL logs for both incremental full backups.  CNGP and Barman require an S3 or Google Cloud Storage interface to save backups. Full backups are configured to run nightly at midnight. Backups are saved to a Weka S3 interface.  Please note that this is same storage location that the database is stored.

The long term backup requirements are to:

- Store backup in physically outside of S3DF?
- Backup every X amount of time?


Monitoring
==========

CNPG has built in Prometheus support for the Pooler and the Database cluster.  The S3DF Prometheus instance scrapes and stores metrics.  Metrics are displayed in the S3DF Grafana.  Metrics will need to be available for <update> days.

The requirements for monitoring are:

- Per Cluster
   - Cluster uptime
   - CPU
   - Memory
      - Available
      - Working Memory
   - Storage
      - used, available overall
      - per database
   - Connections
      - Number of available connections
      - Connections per database
   - Replication and Backup
      - Replication Lag
      - WAL archive failures
      - Successful and Failed backups
   - Indexes
      - Most and least frequently scanned
   - Database activity
      - Rows inserted
      - Rows updated
      - Rows deleted
      - Dead Rows
   - Cache
      - Cache hit rate
- Checks per database
   - Operations
      - Analyze
      - Vaccuum
      - Freeze
      - Bloat
   - Locks
      - Locks by Lock Mode
      - Deadlocks

Logging
=======

CNPG logs to stdout and stderr.  Logs are available via the `kubectl logs` command.  Currently there is not a solution for long term retention of logging.  The options are using Loki, Elasticsearch, or Gooogle Cloud Logging.  Logs will be be available for <update days>

The requirements for logs are:
- Store logs for X days?
- Provide log access to administrators and developers?

See the `reStructuredText Style Guide <https://developer.lsst.io/restructuredtext/style.html>`__ to learn how to create sections, links, images, tables, equations, and more.

.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
