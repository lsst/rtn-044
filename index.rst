:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========

Postgres database design for Butler in the USDF.  Includes details on Postgres architecture in Kubernetes, authentication, backups, and monitoring/logging.  The sections below are all draft and will likely change after design discussions.


Architecture
============

Postgres is deployed on Kubernetes using the CloudNativePG (CNPG) Kuberneter opeator.  CNPG is an open source project developed by Enterprise DB.  The CNPG Operator includes built in capabilities to manage upgrades, high availability, replication, and backup.  Commerical support is available for a fee.

A seperate development Kubernetes cluster and production Kubernetes cluster are deployed.  This allows for testing both operator functionality and configuration prior to deploying in production.

Version
=======

Postgres version 14 is deployed. Previously Postgres 12 was deployed at NCSA and Postgres 13 at IDF.  Postgres 14 has performance improvments and is the latest stable major version.  There are no known limitations that would prevent Butler from running on Postgres 14.

Storage
=======

The storage for the CNGP clusters is on Weka storage using the Container Storage Interface (CSI) driver.  1000 GB is provisioned for production.  Volume expansion is supported to increase the size of the disks.  Both CNGP and the Weka CSI Plugin support for volume expansion.


Access Methods
==============

All end user access to the Butler Postgres database will be through Butler.  No direct access through PSQL or other Postgres administrative tools is required by end users.  Butler Postgres does not require external connectivty outside of USDF so no external IP address is needed on the database.

Postgres administrators will be able to use PSQL and other tools to perform administrative functions.  PSQL can be run through the S3DF Rubin Servers.


Authentication and Access Control
=================================

There are four types of access needed to Butler Postgres.
#. Read only access - Read data through Butler
#.  Developer write access - Write data to Butler
#. PaNDA Service account - Query butler from jobs and store results of job runs
#. Administrative access - Create databases, tables, edit roles

For individual user accounts in Postgres a username needs to be created and role assigned.  


#. Shared Username (rubin) with shared password.

The advantages of this approach are
- There is no support needed to reset passwords
- Easy to deploy and build into image without end user intervention
- Remove risk of an developer creating a production database with only their ownership and schema

#. Individual username with

#. LDAP
LDAPS and LDAP with Start TLS were tested for authentication.  Neither was able to successfully get working.  An unknown error was returned by Postgres.  It also appears that PG Bouncer does not support LDAP based on an open issue in the PG Bouncer GitHub repository.  The following options are available for authentication.

scram-sha-256 will be used for password encryption as is now is the default for Postgres 14.  This encryption method was previously used by Butler in other environments.


Monitoring
==========

CNPG has built in Prometheus support for the Pooler and the Database cluster.  The S3DF Prometheus instance scrapes and stores metrics.  Metrics are displayed in the S3DF Grafana.  Metrics will need to be available for <update> days.

The requirements for monitoring are:
* Cluster uptime
* CPU, Memory usage
* Disk available and storage by database

Logging
=======

CNPG logs to stdout and stderr.  Logs are available via the `kubectl logs` command.  Currently there is not a solution for long term retention of logging.  The options are using Loki, Elasticsearch, or Gooogle Cloud Logging.  Logs will be be available for <update days>

The requirements for logs are:


Add content here
================

Add content here.
See the `reStructuredText Style Guide <https://developer.lsst.io/restructuredtext/style.html>`__ to learn how to create sections, links, images, tables, equations, and more.

.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
