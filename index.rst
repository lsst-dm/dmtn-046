..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.

.. note::

  This tech note contains technology investigations done in the period 2007 to 2013.
  The first three sections were originally published in :cite:`Document-8256` and subsequently :cite:`LDM-135` (`see DocuShare version 18 <https://docushare.lsstcorp.org/docushare/dsweb/Get/Version-24508/LDM-135.pdf>`_).
  `Document LDM-135 <https://ls.st/LDM-135>`_ no longer contains the historical background and the content has been migrated to different locations.
  :ref:`experiments-on-hive-trac` is originally from `LSST Trac`_ and was referenced from LDM-135.


.. _potential-solutions:

Potential Solutions - Research
==============================

The two most promising technologies able to scale to LSST size available
today are Relational Database Management Systems (RDBMS) and Map/Reduce
(MR): the largest DBMS system, based on commercial Teradata and reaching
20+ petabytes is hosted at eBay, and the largest MR-based systems,
reaching many tens of petabytes are hosted at Google, Facebook, Yahoo!
and others.

.. _solutions-research:

The Research
------------

To decide which technology best fits the LSST requirements :cite:`LPM-17`, we did an
extensive market research and analyses, reviewed relevant literature,
and run appropriate stress-tests for selected “promising” candidates,
focusing on weak points and the most uncertain aspects. Market research
and analyses involved (a) discussions about lessons learned with many
industrial and scientific users dealing with large data sets, (b)
discussions about existing solutions and future product road-maps with
leading solution providers and promising start-ups, and (c), research
road-map with leading database researchers from academia. See
:ref:`community-consult`.

.. _solutions-results:

The Results
-----------

As a result of our research, we determined an RDBMS-based solution
involving a shared-nothing parallel database is a much better fit for
the LSST needs than MR. The main reasons are availability of indexes
which are absolutely essential for low-volume queries and spatial
indexes, support for schemas and catalogs, performance and efficient use
of resources.


Even though a suitable open source off-the-shelf DMBS capable of meeting
LSST needs does *not* exist today, there is a good chance a system
meeting most of the key requirements will be available well before LSST
production starts. In particular, there are two very promising
by-products of our research:

- we co-initiated :cite:`2008becla-dsj2`, co-founded, and helped bootstrap SciDB – a new
  open source shared nothing database system, and

- we pushed the development of MonetDB, an open source columnar
  database into directions very well aligned with LSST needs. We
  closely collaborate with the MonetDB team – building on our Qserv
  lessons-learned, the team is trying to add missing features and turn
  their software into a system capable of supporting LSST needs. In
  2012 we demonstrated running Qserv with a MonetDB backend instead of
  MySQL.

Both SciDB and MonetDB have strong potential to become the LSST database
solution once they mature.

Further, our research led to creation a new, now
internationally-recognized conference series, `Extremely Large Databases
(XLDB) <http://www.xldb.org>`_. As we continue leading the `XLDB`_ effort, it
gives us a unique opportunity to reach out to a wide range of
high-profile organizations dealing with large data sets, and raise
awareness of the LSST needs among researchers and developers working on
both MR and DBMS solutions.

The remaining of this chapter discusses lessons learned to-date, along
with a description of relevant tests we have run.

.. _mapreduce-nosql:

Map/Reduce-based and NoSQL Solutions
------------------------------------

Map/Reduce is a software framework to support distributed computing on
large data sets on clusters of computers. Google’s implementation
:cite:`Dean:2008:MSD:1327452.1327492`, believed to be the most advanced, is proprietary, and in
spite of Google being one of the LSST collaborators, we were unable to
gain access to any of their MR software or infrastructure. Additional
Google-internal MR-related projects include BigTable :cite:`Chang:2008:BDS:1365815.1365816`,
Chubby :cite:`Burrows:2006:CLS:1298455.1298487`, and Sawzall :cite:`Pike:2005:SP:962135`.
BigTable addresses the need for rapid
searches through a specialized index; Chubby adds transactional support;
and Sawzall is a procedural language for expressing queries. Most of
these solutions attempt to add partial database-like features such as
schema catalog, indexes, and transactions. The most recent MR
developments at Google are Dremel :cite:`Melnik:2010:DIA:1920841.1920886` - an interactive ad-hoc query
system for analysis of read-only data, and Tenzing – a full SQL
implementation on the MR Framework :cite:`Chattopadhyay:2011:37200`. [#]_

.. [#] Through our `XLDB`_ efforts, Google provided us with a
   preprint of a Tenzing manuscript accepted for publication at VLDB 2011.

In parallel to the closed-source systems at Google, similar open-source
solutions are built by a community of developers led by Facebook, Yahoo!
and Cloudera, and they have already gained wide-spread acceptance and
support. The open source version of MR, *Hadoop*, has became popular in
particular among industrial users. Other solutions developed on top (and
“around”) Hadoop include `HBase`_ (equivalent of BigTable), `Hive`_
(concept similar to Google's Dremel), *Pig Latin* (equivalent to
Google's Sawzall), `Zookeeper`_ (equivalent to Google's Chubby), *Simon*,
and others. As in Google's case, the primary purpose of building these
solutions is adding database-features on top of MR. Hadoop is
commercially supported by Cloudera, `Hortonworks`_ [Yahoo] and
`Hadapt`_.

We have experimented with Hadoop (0.20.2) and Hive (0.7.0) in mid 2010
using a 1 billion row USNO-B data set on a 64 node cluster
(see :ref:`experiments-on-hive-trac` for more details). Common LSST
queries were tested, ranging from low-volume type (such as finding a
single object, selecting objects near other know object), through
high-volume ones (full table scans) to complex queries involving joins
(join was implemented in a standard way, in the *reduce* step). The
results were discussed with Hadoop/`Hive`_ experts from Cloudera.
Periodically we revisit the progress and feature set available in the
Hadoop ecosystem, but to date we have not found compelling reasons to
consider Hadoop as a serious alternative for managing LSST data.

Independently, Microsoft developed a system called `Dryad`_, geared
towards executing distributed computations beyond “flat” *Map* and
*Reduce*, along with a corresponding language called *LINQ*. Due to its
strong dependence on Windows OS and limited availability, use of `Dryad`_
outside of Microsoft is very limited. Based on news reports :cite:`Foley:2011:Zdnet`,
Microsoft dropped support for `Dryad`_ back in late 2011.

Further, there is a group of new emerging solutions often called as
*NoSQL*. The two most popular ones are `MongoDB`_ and `Cassandra`_.

The remaining of this section discusses all of the above-mentioned
products.

Further details about individual MR and no-SQL solutions can be found in
:ref:`mr-solutions` and :ref:`db-solutions`.

.. _dbms-solutions:

DBMS Solutions
--------------

Database systems have been around for much longer than MR, and therefore
they are much more mature. They can be divided into many types:
parallel/single node, relational/object-oriented, columnar/row-based;
some are built as appliances. Details about individual DBMS products and
solutions we considered and/or evaluated can be found in
:ref:`db-solutions`.

.. _parallel-dbms:

Parallel DBMSes
~~~~~~~~~~~~~~~

Parallel databases, also called MPP DBMS (massively parallel processing
DBMS), improve performance through parallelization of queries: using
multiple CPUs, disks and servers in parallel. Data is processed in
parallel, and aggregated into a final result. The aggregation may
include computing average, max/min and other aggregate functions. This
process is often called *scatter-gather*, and it is somewhat similar to
*map* and *reduce* stages in the MR systems.

Shared-nothing parallel databases, which fragment data and in many cases
use an internal communications strategy similar to MR, scale
significantly better than single-node or shared-disk databases. Teradata
uses proprietary hardware, but there are a number of efforts to leverage
increasingly-fast commodity networks to achieve the same performance at
much lower cost, including Greenplum, DB2 Parallel Edition, Aster Data,
GridSQL, ParAccel, InfiniDB, SciDB, and Project Madison at Microsoft
(based on DATAllegro, acquired by Microsoft in 2008). Most of these
efforts are relatively new, and thus the products are relatively
immature. EBay's installation used to be based on Greenplum in 2009 and
reached 6.5 PB, but their current Singularity system is now approaching
30 PB and is based on Teradata's appliances. Some of these databases
have partition-wise join, which can allow entity/observation join
queries to execute more efficiently, but none allow overlapping
partitions, limiting the potential performance of pairwise analysis.

Microsoft SQL Server offers Distributed Partitioned Views, which provide
much of the functionality of a shared-nothing parallel database by
federating multiple tables across multiple servers into a single view.
This technology is used in the interesting GrayWulf project :cite:`Szalay:2008:Graywulf`
:cite:`Simmhan:2009:4755781` which is designed to host observational data consisting of
Pan-STARRS PS1 :cite:`2007IAUS..236..341J` astronomical detections and summary information
about the objects that produced them. GrayWulf partitions observation
data across nodes by “zones” :cite:`Gray:2006:Zones`, but these partitions cannot overlap.
Fault tolerance is built in by having three copies of the data, with one
undergoing updates – primarily appending new detections – and the other
two in a hot/warm relationship for failover. GrayWulf has significant
limitations, however. The object information for the Pan-STARRS PS1 data
set is small enough (few TB) that it can be materialized on a single
node. The lack of partition-wise join penalizes entity/observation join
queries and pairwise analysis. The overall system design is closely tied
to the commercial SQL Server product, and re-hosting it on another
RDBMS, in particular an open source one, would be quite difficult.

The MPP database is ideal for the LSST database architecture.
Unfortunately, the only scalable, proven off-the-shelf solutions are
commercial and expensive: Teradata, Greenplum. Both systems are (or
recently were) behind today world's largest production database systems
at places such as eBay :cite:`Monash:2009:ebay` :cite:`Monash:2010:ebay` and Walmart :cite:`Schuman:2004:eWeek`.
IBM's DB2 “parallel edition”, even though it implements a shared-nothing
architecture since mid-1990 focuses primarily on supporting unstructured
data (XML), not large scale analytics.

The emergence of several new startups, such as Aster Data, DataAllegro,
ParAccel, GridSQL and SciDB is promising, although some of them have
already been purchased by the big and expensive commercial RDBMSes:
Teradata purchased Aster Data, Microsoft purchased DataAllegro. To date,
the only shared-nothing parallel RDBMS available as open source is SciDB
– its first production version (*v11.06*) was released in June 2011.
ParAccel is proprietary, we did not have a chance to test it, however
given we have not heard of any large scale installation based on
ParAccel we have doubts whether it'll meet our needs. After testing
GridSQL we determined it does not offer enough benefits to justify using
it, the main cons include limited choices of partitioning types (hash
partitioning only), lack of provisions for efficient near neighbor
joins, poor stability and lack of good documentation.

SciDB :cite:`2009:Cudre-Mauroux:DSS:1687553.1687584` is the only parallel open source DBMS currently available on the
market. It is a columnar, shared-nothing store based on an array data
model. The project has been inspired by the `LSST needs <http://web.archive.org/web/20120731061725/www.scidb.org/about/history.php>`_, and the
LSST Database team is continuously in close communication with the SciDB
developers. SciDB’s architectural features of chunking large arrays into
overlapping chunks and distributing these chunks across a shared nothing
cluster of machines match the LSST database architecture. Initial tests
run with the v0.5 SciDB release exposed architectural issues with SciDB
essential for LSST, related to clustering and indexing multi-billion,
sparse arrays of objects in a 2-dimensional (ra, declination) space.
These issues have been addressed since then and re-testing is planned.

There are several reasons why SciDB is not our baseline, and we
currently do not have plans to use it for LSST catalog data. First, as
an array database, SciDB uses a non-SQL query language (actually, two)
appropriate for arrays. Adapting this to SQL, likely through a
translation layer, is a substantial burden, even more difficult than
parsing SQL queries for reissue as other SQL queries. (Given the
widespread use of SQL in the astronomy community and the ecosystem of
tools available for SQL, moving away from SQL would be a major
endeavor.) Second, while relations can be thought of as one-dimensional
arrays, SciDB is not optimized to handle them as well as a traditional
RDBMS, in particular for the variety of joins required (including star
schema, merge joins, and self joins). Standard RDBMS features like
views, stored procedures, and privileges would have to be added from the
ground up. Third, SciDB's fault tolerance is not yet at the level of
`XRootD`_. Overall, the level of coding we would have to do to build on the
current SciDB implementation appears to be larger than what we are
planning on top of `XRootD`_/MySQL. As SciDB's implementation progresses,
though, this trade-off could change.

.. _object-oriented-solution:

Object-oriented solutions
~~~~~~~~~~~~~~~~~~~~~~~~~

The object-oriented database market is very small, and the choices are
limited to a few small proprietary solutions, including Objectivity/DB
and InterSystems Caché. Objectivity/DB was used by the BaBar experiment
in 1999 – 2002, and the BaBar database reached a petabyte :cite:`DBLP:conf/cidr/BeclaW05`. The
members of LSST database team, being the former members of the BaBar
database team are intimately familiar with the BaBar database
architecture. The Objectivity/DB was used primarily as a simple data
store, all the complexity, including custom indices had to be all built
in custom, user code. Given that, combining with the challenges related
to porting and debugging commercial system led as to a conclusion
Objectivity/DB is not the right choice for LSST.

InterSystems Caché
`has been chosen as <http://www.intersystems.com/library/library-item/european-space-agency-chooses-intersystems-cach-database-for-gaia-mission-to-map-milky-way/>`_
the underlying system for the European Gaia project :cite:`Zicaro:2011:ODBMS` :cite:`2016A&A..595A..1G`,
based on our limited knowledge, so far
the Gaia project focused primarily on using Caché for ingest-related
aspects of the system, and did not have a chance to research analytical
capabilities of Caché at scale.

.. _row-vs-columnar:

Row-based vs columnar stores
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Row-based stores organize data on disk as rows, while columnar store –
as columns. Column-store databases emerged relatively recently, and are
based on the C-store work :cite:`Stonebraker:2005:CCD:1083592.1083658`. By operating on columns
rather than rows, they are able to retrieve only the columns required
for a query and greatly compress the data within each column. Both
reduce disk I/O and hence required hardware by a significant factor for
many analytical queries on observational data that only use a fraction
of the available columns. Current column stores also allow data to be
partitioned across multiple nodes, operating in a shared-nothing manner.
Column stores are less efficient for queries retrieving small sets of
full-width records, as they must reassemble values from all of the
columns.

Our baseline architecture assumes all large-volume queries will be
answered through shared scans, which reduces wasting disk I/O for
row-based stores: multiple queries attached to the same scan will
typically access most columns (collectively). We are also vertically
partitioning our widest table into frequently-accessed and
infrequently-accessed columns to get some of the advantage of a column
store.

Nevertheless, a column store could still be more efficient. Work done at
Google (using Dremel) has claimed that “the crossover point often lies
at dozens of fields but it varies across data sets” :cite:`Melnik:2010:DIA:1920841.1920886`. In our case,
the most frequently accessed table: Object, will have over “20 dozens”
columns. The Source, DiaObject, and DiaSource tables will each have
about 4 dozen columns. These could be wide enough that all
simultaneously executing queries will still only touch a subset of the
columns. Most other large tables are relatively narrow and are expected
to have all columns used by every query. Low query selectivity (expected
to be <1% for full table scans) combined with late materialization
(postponing row assembly until the last possible moment) is expected to
further boost effectiveness of columnar stores.

The two leading row-based DBMSes are MySQL and PostgreSQL. Of these two,
MySQL is better supported, and has much wider community of users,
although both are commercially supported (MySQL: Oracle,
MontyProgram+SkySQL, Percona. PostgreSQL: EnterpriseDB). PostgreSQL
tends to focus more on OLTP, while MySQL is closer to our analytical
needs, although both are weak in the area of scalability. One of the
strongest points of PostgreSQL used to be spatial GIS support, however
MySQL has recently rewritten their GIS modules and it now offers true
spatial relationship support (starting from version 5.6.1). Neither
provides good support for spherical geometry including wraparound,
however.

Many commercial row-bases DBMSes exist, including Oracle, SQL Server,
DB2, but they do not fit well into LSST needs, since we would like to
provide all scientists with the ability to install the LSST database at
their institution at low licensing and maintenance cost.

Columnar stores are starting to gain in popularity. Although `the list
is already relatively large
<https://en.wikipedia.org/wiki/Column-oriented_DBMS>`_, the number of
choices worth considering is relatively small. Today's most popular
commercial choice is HP Vertica, and the open source solutions include
MonetDB :cite:`Ivanova:2007:4274958` :cite:`DBLP:journals/debu/IdreosGNMMK12`
and Calpont's InfiniDB. The latter also implements shared
nothing MPP, however the multi-server version is only available as part
of the commercial edition.

With help from Calpont, we evaluated InfiniDB and demonstrated it could
be used for the LSST system – we run the most complex (near neighbor)
query. Details are available in :cite:`DMTN-047`.

We are working closely with the MonetDB team, including the main
architect of the system, Martin Kersten and two of his students who
worked on porting MonetDB to meet LOFAR database needs. In 2011 the
MonetDB team has run some basic tests using astronomical data (USNOB as
well as our DC3b-PT1.1 data set :cite:`Document-9044`). During the course of testing our
common queries they implemented missing features such as support for
user defined functions, and are actively working on further extending
MonetDB to build remaining missing functionality, in particular ability
to run as a shared-nothing system. To achieve that, existing MonetDB
server (*merovingian*) has to be extended. Table partitioning and
overlaps (on a single node) can be achieved through table views,
although scalability to LSST sizes still needs to be tested. Cross-node
partitioning requires new techniques, and the MonetDB team is actively
working on it.

In 2012 with help from the MonetDB team we demonstrated a limited set of
queries on a Qserv system integrated with MonetDB on the backend rather
than MySQL.\ [*]_ While the integration was left incomplete, the speed at
which we were able to port Qserv to a new database and execute some
queries is convincing evidence of Qserv's modularity. Because basic
functionality was ported in one week, we are confident that porting to
another DBMS can be done with modest effort in a contingency or for
other reasons. The experience has also guided Qserv design directions
and uncovered unintended MySQL API dependence in Qserv and broader LSST
DM systems.

.. _appliances:

Appliances
~~~~~~~~~~

Appliances rely on specialized hardware to achieve performance. In
general, we are skeptical about appliances, primarily because they are
locking us into this specialized hardware. In addition, appliances are
usually fast, however their replacement cost is high, so often commodity
hardware is able to catch up, or even exceed the performance of an
appliance after few years (the upgrade of an appliance to a latest
version is usually very costly).

.. _solution-comparison-discussion:

Comparison and Discussion
-------------------------

The MR processing paradigm became extremely popular in the last few
years, in particular among peta-scale industrial users. Most industrial
users with peta-scale data sets heavily rely on it, including places
such as Google, Yahoo!, Amazon or Facebook, and even eBay has recently
started using Hadoop for some of their (offline, batch) analysis. The
largest (peta-scale) RDBMS-based systems all rely on shared-nothing, MPP
technology, and almost all on expensive Teradata solutions (eBay,
Walmart, Nokia, for a few years eBay used Greenplum but they switched
back to Teradata's Singularity).

In contrast, science widely adopted neither RDBMS nor MR. The community
with the largest data set, HEP, is relying on a home-grown system,
augmented by a DBMS (typically Oracle or MySQL) for managing the
metadata. This is true for most HEP experiments of the last decade (with
the exception of BaBar which initially used Objectivity), as well as the
LHC experiments. In astronomy, most existing systems as well as the
systems starting in the near future are RDBMS-based (SDSS – SQL Server,
Pan-STARRS – SQL Server, 2MASS – Informix, DES – Oracle, LOFAR –
MonetDB, Gaia – Caché). It is worth noting that none of these systems
was large enough so far to break the single-node barrier, with the
exception of Pan-STARRS. Geoscience relies primarily on netCDF/HDF5
files with metadata in a DBMS. Similar approach is taken by bio
communities we have talked to. In general, MR approach has not been
popular among scientific users so far.

The next few sections outline key differences, strengths and weaknesses
of MR and RDBMS, and the convergence.

.. _comparison-apis:

APIs
~~~~

In the MR world, data is accessed by a pair of functions, one that is
“mapped” to all inputs, and one that “reduces” the results from the
parallel invocations of the first. Problems can be broken down into a
sequence of MR stages whose parallel components are explicit. In
contrast, a DBMS forces programmers into less natural, declarative
thinking, giving them very little control over the flow of the query
execution; this issue might partly go away by interacting with database
through a user defined function (UDFs), which are becoming increasingly
popular. They must trust the query optimizer's prowess in “magically”
transforming the query into a query *plan*. Compounding the difficulty
is the optimizer's unpredictability: even one small change to a query
can make its execution plan efficient or painfully slow.

The simplicity of the MR approach has both advantages and disadvantages.
Often a DBMS is able to perform required processing on the data in a
small number of passes (full table scans). The limited MR operators on
the other hand may lead to many more passes through the data, which
requires more disk I/O thus reduces performance and increases hardware
needed. Also, MR forced users to code a lot of operations typically
provided by an RDBMS *by-hand* – these include joins, custom indexes or
even schemas.

.. _comparison-scability:

Scalability, fault tolerance and performance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The simple processing framework of MR allows to easily, incrementally
scale the system out by adding more nodes as needed. Frequent
check-pointing done by MR (after every “map” and every “reduce” step)
simplifies recoverability, at the expense of performance. In contrast,
databases are built with the optimistic assumptions that failures are
rare: they generally checkpoint only when necessary. This has been shown
through various studies :cite:`Pavlo:2009:CAL:1559845.1559865`.

The frequent checkpointing employed by MR, in combination with limited
set of operators discussed earlier often leads to inefficient usages of
resources in MR based systems. Again, this has been shown through
various studies. EBay's case seems to support this as well: back in 2009
when they managed 6.5 petabytes of production data in an RDBMS-based
system they relied on a mere 96 nodes, and based on discussions with the
original architects of the eBay system, to achieve comparable processing
power through MR, many thousand nodes would be required.

.. _comparison-flexibility:

Flexibility
~~~~~~~~~~~

MR paradigm treats a data set as a set of key-value pairs. It is
structure-agnostic, leaving interpretation to user code and thus
handling both poorly-structured and highly-complex data. Loose
constraints on data allow users to get to data quicker, bypassing schema
modeling, complicated performance tuning, and database administrators.
In contrast, data in databases are structured strictly in records
according to well-defined schemata.

While adjusting schema with ease is very appealing, in large scientific
projects like LSST, the schema has to be carefully thought through to
meet the needs of many scientific collaborations, each having a
different set of requirements. The flexibility would be helpful during
designing/debugging, however it is of lesser value for a science
archive, compared to industry with rapidly changing requirements, and a
strong focus on agility.

.. _comparison-cost:

Cost
~~~~

As of now, the most popular MR framework, *Hadoop*, is freely available
as open source. In contrast, none of the freely available RDBMSes
implements a shared-nothing MPP DBMS (to date), with the exception of
SciDB, which can be considered only partially relational.

From the LSST perspective, plain MR does not meet project's need, in
particular the low-volume query short response time needs. Significant
effort would be required to alleviate Hadoop's high latency (today's
solution is to run idle MR daemons, and attach jobs to them, which
pushes the complexity of starting/stopping jobs onto user code). Also,
table joins, typically done in *reduce* stage, would have to be
implemented as *maps* to avoid bringing data for joined tables to
Reducer – in practice this would require implementing a clever data
partitioning scheme. The main advantages of using MR as a base
technology for the LSST system include scalability and fault-tolerance,
although as alluded above, these features come at a high price:
inefficient use of resources (full checkpointing between each *Map* and
each *Reduce* step), and triple redundancy.

.. _comparison-summary:

Summary
~~~~~~~

The key features of an ideal system, along with the comments for both
Map/Reduce and RDBMS are given in the table below.

.. table::

   +-----------------------+---------------------+------------------------------+
   | Feature               | Map/Reduce          | RDBMS                        |
   +=======================+=====================+==============================+
   | Shared nothing, MPP,  | Implements it.      | Some implement it, but only  |
   | columnar              |                     | as commercial, non open      |
   |                       |                     | source to date,              |
   |                       |                     | except not-yet-mature SciDB. |
   +-----------------------+---------------------+------------------------------+
   | Overlapping           | Nobody implements   | Only SciDB implements this   |
   | partitions, needed    | this.               | to-date.                     |
   | primarily for         |                     |                              |
   | near-neighbor         |                     |                              |
   | queries               |                     |                              |
   +-----------------------+---------------------+------------------------------+
   | Shared scans          | This kind of logic  | There is a lot of research   |
   | (primarily for        | would have to be    | about shared scans in        |
   | complex queries that  | implemented by us.  | databases. Implemented       |
   | crunch through large  |                     | by Teradata. Some vendors,   |
   | sets of data)         |                     | including SciDB are          |
   |                       |                     | considering implementing it  |
   +-----------------------+---------------------+------------------------------+
   | Efficient use of      | Very inefficient.   | Much better than MR.         |
   | resources             |                     |                              |
   | Catalog/schema        | Started adding      | Much better than in MR.      |
   |                       | support, e.g.,      |                              |
   |                       | `Hive`_, HadoopDB   |                              |
   +-----------------------+---------------------+------------------------------+
   | Indexes (primarily    | Started adding      | Much better than in MR.      |
   | for simple queries    | support, e.g.,      |                              |
   | from public that      | `Hive`_, HadoopDB   |                              |
   | require real time     |                     |                              |
   | response)             |                     |                              |
   +-----------------------+---------------------+------------------------------+
   | Open source           | Hadoop (although it | No shared-nothing MPP        |
   |                       | is implemented in   | available as open source     |
   |                       | Java, not ideal     | yet except still-immature    |
   |                       | from LSST point of  | SciDB. We expect there will  |
   |                       | view)               | be several by the time LSST  |
   |                       |                     | needs it (SciDB, MonetDB,    |
   |                       |                     | ParAccel and others)         |
   +-----------------------+---------------------+------------------------------+

.. _convergence:

Convergence
~~~~~~~~~~~

Despite their differences, the database and MR communities are learning
from each other and seem to be converging.

The MR community has recognized that their system lacks built-in
operators. Although nearly anything can be implemented in successive MR
stages, there may be more efficient methods, and those methods do not
need to be reinvented constantly. MR developers have also explored the
addition of indexes, schemas, and other database-ish features.\ [#]_
Some have even built a complete relational database system\ [#]_ on top
of MR.

.. [#] An example of that is `Hive`_.

.. [#] An example of that is `HadoopDB <http://db.cs.yale.edu/hadoopdb/hadoopdb.html>`_

The database community has benefited from MR's experience in two ways:

1. Every parallel shared-nothing DBMS can use the MR execution style
   for internal processing – while often including more-efficient
   execution plans for certain types of queries. Though systems such as
   Teradata or IBM's DB2 Parallel Edition have long supported this, a
   number of other vendors are building new shared-nothing-type
   systems.\ [#]_ It is worth noting that these databases typically use
   MR-style execution for aggregation queries.

2. Databases such as Greenplum (part of EMC) and Aster Data (part of
   Teradata since March 2011) have begun to explicitly support the MR
   programming model with user-defined functions. DBMS experts have
   noted that supplying the MR programming model on top of an existing
   parallel flow engine is easy, but developing an efficient parallel
   flow engine is very hard. Hence it is easier for the DMBS community
   to build map/reduce than for the map/reduce community to add full
   DBMS functionality.

.. [#] ParAccel, Vertica, Aster Data, Greenplum, DATAllegro (now part of Microsoft), Datapuia, Exasol and SciDB

The fact MR community is rapidly adding database/SQL like features on
top of their plain MR (Tenzing, `Hive`_, HadoopDB, etc), confirms the need
for database-like features (indexes, schemas, catalogs, sql).

As we continue monitoring the latest development in both RDBMS and MR
communities and run more tests, we expect to re-evaluate our choices as
new options become available.

.. FIXME look for footnotes

.. _mr-solutions:

Map/Reduce Solutions
====================

.. _mr-hadoop:

Hadoop
------

Hadoop is a Lucene sub-project hosted by Apache. It is open source. It
tries to re-create the Google MR technology :cite:`Dean:2008:MSD:1327452.1327492` to provide a
framework in which parallel searches/projections/transformations (the
*Map* phase) and aggregations/groupings/sorts/joins (the Reduce phase)
using key-value pairs can be reliably performed over extremely large
amounts of data. The framework is written in Java though the actual
tasks executing the map and reduce phases can be written in any language
as these are scheduled external jobs. The framework is currently
supported for GNU/Linux platforms though there is on-going work for
Windows support. It requires that ssh be uniformly available in order to
provide daemon control.

Hadoop consists of over 550 Java classes implementing multiple
components used in the framework:

- The Hadoop Distributed File System (HDFS), a custom POSIX-like file
  system that is geared for a write-once-read-many access model.  HDFS
  is used to distribute blocks of a file, optionally replicated, across
  multiple nodes. HDFS is implemented with a single Namenode that
  maintains all of the meta-data (i.e., file paths, block maps, etc.)
  managed by one or more Datanodes (i.e., a data server running on each
  compute node). The Namenode is responsible for all meta-data
  operations (e.g., renames and deletes) as well as file allocations.
  It uses a rather complicated distribution algorithm to maximize the
  probability that all of the data is available should hardware failures
  occur. In general, HDFS always tries to satisfy read requests with
  data blocks that are closest to the reader. To that extent, HDFS also
  provides mechanisms, used by the framework, to co-locate jobs and
  data. The HDFS file space is layered on top of any existing native
  file system.

- A single JobTracker, essentially a job scheduler responsible for
  submitting and tracking map/reduce jobs across all of the nodes.

- A TaskTracker co-located with each HDFS DataNode daemon which is
  responsible for actually running a job on a node and reporting its
  status.

- DistributedCache to distribute program images as well as other
  required read-only files to all nodes that will run a map/reduce
  program.

- A client API consisting of JobConf, JobClient, Partitioner,
  OutputCollector, Reporter, InputFormat, OutputFormat among others that
  is used to submit and run map/reduce jobs and retrieve the output.

Hadoop is optimized for applications that perform a streaming search
over large amounts of data. By splitting the search across multiple
nodes, co-locating each with the relevant data, wherever possible, and
executing all the sub-tasks in parallel, results can be obtained
(relatively) quickly. However, such co-ordination comes with a price.
Job setup is a rather lengthy process and the authors recommend that the
map phase take at least a minute to execute to prevent job-setup from
dominating the process. Since all of the output is scattered across many
nodes, the map phase must also be careful to not produce so much output
as to overwhelm the network during the reduce phase, though the
framework provides controls for load balancing this operation and has a
library of generally useful mappers and reducers to simplify the task.
Even so, running ad hoc map/reduce jobs can be problematic. The latest
workaround used by many Hadoop users involves running Hadoop services
continuously (and jobs are attached to these services very fast). By
default, joining tables in MR involves transferring data for all the
joined tables into the *reducer*, and performing the join in the
*reduce* stage, which can easily overwhelm the network. To avoid this,
data must be partitioned, and data chunked joined together must be
placed together (on the same node), in order to allow performing the
join in the *map* stage.

Today's implementation of Hadoop requires full data scan even for
simplest queries. To avoid this, indices are needed. Implementing
indices has been planned by the community for several years, and
according to the latest estimates they will be implemented in one or two
years. In the meantime, those who need indices must implement and
maintain them themselves, the index files can be stored e.g. as files in
the Hadoop File System (HDFS).

One of the “features” of MR systems is lack of official catalog
(schema); instead, knowledge about schema in part of the code. While
this dramatically improves flexibility and speeds up prototyping, it
makes it harder to manage such data store in the long term, in
particular if multi-decade projects with large number of developers are
involved.

Lack of some features that are at the core of every database system
should not be a surprise – MR systems are simply built with different
needs in mind, and even `the Hadoop website officially states that
*Hadoop is not a substitute for a database*
<https://wiki.apache.org/hadoop/HadoopIsNot>`_. Nethertheless, many have
attempted to compare Hadoop performance with databases. According to
some publications and feedback from Hadoop users we talked to, Hadoop is
about an order of magnitude more wasteful of hardware than a e.g. DB2
:cite:`DeWitt:2008:MapReduce` :cite:`DeWitt:2008:MapReduceII`.

Hadoop has a large community supporting it; e.g., over 300 people
attended the first Hadoop summit (in 2008). It is used in production by
`many organizations <https://wiki.apache.org/hadoop/PoweredBy>`_,
including Facebook, Yahoo!, and Amazon Facebook.
It is also commercially supported by Cloudera. Hadoop Summit 2011 was
attended by more than 1,600 people from more than 400 companies.

We evaluated Hadoop in 2008. The evaluation included discussions with
key developers, including Eric Baldeschwieler from Yahoo!, Jeff
Hammerbacher from Facebook, and later Cloudera, discussions with users
present at the 1\ :sup:`st` Hadoop Summit, and a meeting with the
Cloudera team in September of 2010.

.. _mr-hive:

Hive
----

`Hive`_ is a data warehouse infrastructure developed by Facebook on
top of Hadoop; it puts structures on the data, and defines SQL-like
query language. It inherits Hadoop's deficiencies including high latency
and expensive joins. `Hive`_ works on static data, it particular it can't
deal with changing data, as row-level updates are not supported.
Although it does support some database features, it is a “state where
databases were ~1980: there are almost no optimizations” (based on
Cloudera, meeting at SLAC Sept 2010). Typical solutions involve
implementing missing pieces in user code, for example once can build
their own indexes and interact directly with HDFS (and skip the Hadoop
layer).

.. _mr-hbase:

HBase
-----

`HBase`_ is a column-oriented structured storage modeled after Google's
Bigtable :cite:`Chang:2008:BDS:1365815.1365816`, and built on top of the Hadoop HDFS. It is good
at incremental updates and column key lookups, however, similarly to
plain MR, it offers no mechanism to do joins – a typical solution used
by most users is to denormalize data. `HBase`_ is becoming increasingly
more popular at Facebook :cite:`Peschka:2010:HBase`. It is supported commercially by
Cloudera, Datameer and `Hadapt`_.

.. _mr-pig-latin:

Pig Latin
---------

Pig Latin is a procedural data flow language for expressing data
analysis programs. It provides many useful primitives including filter,
foreach ... generate, group, join, cogroup, union, sort and distinct,
which greatly simplify writing Map/Reduce programs or gluing multiple
Map/Reduce programs together. It is targeted at large-scale
summarization of datasets that typically require full table scans, not
fast lookups of small numbers of rows. We have talked to the key
developer of Pig Latin – Chris Olston.

.. _mr-other:

Other Hadoop-related Systems
----------------------------

Other systems build for Hadoop include `Zookeeper`_ – a service for
coordinating Hadoop's processes (ala Google's Chubby :cite:`Burrows:2006:CLS:1298455.1298487`) , and
Simon – a cluster and application monitoring tool. Simon is similar to
Ganglia, except it has more/better aggregation.

.. _mr-dryad:

Dryad
-----

`Dryad`_ :cite:`Isard:2007:DDD:1272996.1273005` is a system developed by Microsoft Research for executing
distributed computations. It supports a more general computation model
than MR in that it can execute graphs of operations, using so called
Directed Acyclic Graph (DAG). It is somewhat analogous to the MR model
in that it can model MR itself, among others, more complex flows. The
graphs are similar to the query plans in a relational database. The
graph execution is optimized to take advantage of data locality if
possible, with computation moving to the data. If non-local data is
needed, it is transferred over the network.

`Dryad`_ currently works on flat files. It is similar to Hadoop in this
way.

The core execution engine in `Dryad`_ has been used in production for
several years but not heavily. There are several integration pieces we
might want (loading data from databases instead of files, tracking
replicas of data) that do not yet exist.

Beyond the execution engine, `Dryad`_ also incorporates a simple per-node
task scheduler inherited from elsewhere in Microsoft. It runs
prioritized jobs from a queue. `Dryad`_ places tasks on nodes based on the
data available on the node and the state of the task queue on the node.
A centralized scheduler might improve things, particularly when multiple
simultaneous jobs are running; that is an area that is being
investigated.

`Dryad`_ requires that the localization or partitioning of data be exposed
to it. It uses a relatively generic interface to obtain this metadata
from an underlying filesystem, enabling it to talk to either a
proprietary GFS-like filesystem or local disks.

`Dryad`_ runs only on Windows .NET at present. Building the system outside
of Microsoft is difficult because of dependencies on internal libraries;
this situation is similar to the one with Google's GFS and Map/Reduce.
The core execution engine could conceivably be implemented within Hadoop
or another system, as its logic is not supposed to be terribly
complicated. The performance-critical aspect of the system is the
transfer of data between nodes, a task that Windows and Unix filesystems
have not been optimized for and which `Dryad`_ therefore provides.

`Dryad`_ has been released as open source to academics/researchers in
July 2009. This release however does not include any distributed
filesystem for use with the system. Internally, Microsoft uses the
`Cosmos file system <http://www.goland.org/whatiscosmos/>`_, but it is
not available in the academic release. Instead there are bindings for
NTFS and SQL Server.

Microsoft dropped supporting `Dryad`_ back in late 2011 :cite:`Foley:2011:Zdnet`.

.. _mr-dremel:

Dremel
------

Dremel is a scalable, interactive ad-hoc query system for analysis of
read-only data, implemented as an internal project at Google :cite:`Melnik:2010:DIA:1920841.1920886`.
Information about Dremel has been made available in July 2010. Dremel's
architecture is in many ways similar to our baseline architecture
(executing query in parallel on many nodes in shared nothing
architecture, auto fail over, replicating hot spots). Having said that,
we do not have access to the source code, even though Google is an LSST
collaborator, and there is `no corresponding open source alternative to
date <https://www.quora.com/How-will-Googles-Dremel-change-future-Hadoop-releases>`_.

.. _mr-tenzing:

Tenzing
-------

Tenzing is an SQL implementation on the MapReduce Framework
:cite:`Chattopadhyay:2011:37200` We managed to obtain access to pre-published paper
from Google through our `XLDB`_ channels several months before planned
publication at the VLDB 2011 conference.

Tenzing is a query engine built on top of MR for ad hoc analysis of
Google data. It supports a mostly complete SQL implementation (with
several extensions) combined with several key characteristics such as
heterogeneity, high performance, scalability, reliability, metadata
awareness, low latency support for columnar storage and structured data,
and easy extensibility.

The Tenzing project underscores importance and need of database-like
features in any large scale system.

.. _nosql:

"NoSQL"
-------

The popular term *NoSQL* originally refered to systems that do not
expose SQL interface to the user, and it recently evolved and refers to
structured systems such as key-value stores or document stores. These
systems tend to provide high availability at the cost of relaxed
consistency (“eventual” consistency). Today's key players include
`Cassandra`_ and `MongoDB`_.

While a key/value store might come handy in several places in LSST,
these systems do not address many key needs of the project. Still, a
scalable distributed key-value store may be appropriate to integrate as
an indexing solution within a larger solution.


.. _db-solutions:

Database Solutions
==================

In alphabetical order.

.. _sec-actian:

Actian
------

Actian, `formerly known as Ingres <https://web.archive.org/web/20110925211157/http://www.actian.com/ingres-becomes-actian>`_
provides analytical services
through Vectorwise, acquired from CWI in 2010. Primary speed ups rely on
exploiting data level parallelism (rather than tuple-at-a-time
processing). Main disadvantage from LSST perspective: it is a
single-node system.

.. _sec-cache:

Caché
-----

InterSystems Caché is a shared-nothing object database system, released
as an embedded engine since 1972. Internally it stores data as
multi-dimensional arrays, and interestingly, supports overlaps. We are
in discussions with the company—we have been discussing Caché with
Stephen Angevine since early 2007, and met with Steven McGlothlin in
June 2011. We also discussed Caché with William O'Mullane from the ESA's
Gaia mission, an astronomical survey that selected Caché as their
underlying database store in 2010 [25, 26]). InterSystems offers free
licensing for all development and research, for academic and non-profit
research, plus support contracts with competitive pricing. However,
their system does not support compression and stores data in strings,
which may not be efficient for LSST catalog data.

A large fraction of the code is already available as open source for
academia and non-profit organizations under the name “Globals” :cite:`Intersystems:2008:Globals`.

.. _sec-citrusdb:

CitusDB
-------

`CitusDB`_ is a new commercial distributed database built on top on
PostgreSQL. It supports joins between one large and multiple small
tables (star schema) – this is insufficient for LSST.

.. _db2:

DB2
---

IBM's DB2 “parallel edition” implements a shared-nothing architecture
since mid-1990. Based on discussions with IBM representatives including
Guy Lohman (e.g., at the meeting in June 2007) as well as based on
various news, it appears that IBM's main focus is on supporting
unstructured data (XML), not large scale analytics. All their majors
projects announced in the last few years seem to confirm them, including
Viper, Viper2 and Cobra (XML) and pureScale (OLTP).

.. _db-drizzle:

Drizzle
-------

`Drizzle`_ is a fork from the MySQL Database, the fork was done
shortly after the announcement of the acquisition of MySQL by Oracle
(April 2008). `Drizzle`_ is lighter than MySQL: most advanced features such
as partitioning, triggers and many others have been removed (the code
base was trimmed from over a million lines down to some 300K, it has
also been well modularized). `Drizzle`_'s main focus is on the cloud
market. It runs on a single server, and there are no plans to implement
shared-nothing architecture. To achieve shared-nothing architecture,
`Drizzle`_ has hooks for an opaque sharding key to be passed through
client, proxy, server, and storage layers, but this feature is still
under development, and might be limited to hash-based sharding.

Default engine is InnoDB. MyISAM engine is not part of `Drizzle`_, it is
likely MariaDB engine will become a replacement for MyISAM.

`Drizzle`_\ ’s first GA release occurred in March 2011.

We have discussed the details of `Drizzle`_ with key `Drizzle`_ architects and
developers, including Brian Aker (the chief architect), and most
developers and users present at the `Drizzle`_ developers meeting in April
2008.

.. note::

  In 2017 Drizzle is no longer being developed:
  https://en.wikipedia.org/wiki/Drizzle_(database_server) and
  the Drizzle web site no longer operates.

.. _sec-greenplum:

Greenplum
---------

Greenplum is a commercial parallelization extension of PostgreSQL. It
utilizes a shared-nothing, MPP architecture. A single Greenplum database
image consists of an array of individual databases which can run on
different commodity machines. It uses a single Master as an entry point.
Failover is possible through mirroring database segments. According to
some, it works well with simple queries but has issues with more complex
queries. Things to watch out for: distributed transaction manager,
allegedly there are some issues with it.

Up until recently, Greenplum powered one of the largest (if not the
largest) database setups: eBay was using it to manage 6.5 petabytes of
data on a 96-node cluster :cite:`Monash:2009:ebay`. We are in close contact with the
key eBay developers of this system, including Oliver Ratzesberger.

We are in contact with the Greenplum CTO: Luke Lonergan.

08/28/2008: Greenplum announced supporting MapReduce :cite:`Waas:2009:97836420342207`.

Acquired by EMC in July 2010.

.. _gridsql:

GridSQL
-------

GridSQL is an open source project sponsored by EnterpriseDB. GridSQL is
a thin layer written on top of postgres that implemented shared-nothing
clustered database system targeted at data warehousing. This system
initially looked very promising, so we evaluated it in more details,
including installing it on our 3-node cluster and testing its
capabilities. We determined that currently in GridSQL, the only way to
distribute a table across multiple nodes is via hash partitioning. We
can't simply hash partition individual objects, as this would totally
destroy data locality, which is essential for spatial joins. A
reasonable workaround is to hash partition entire declination zones
(hash partition on zoneId), this will insure all objects for a
particular zone end up on the same node. Further, we can “chunk” each
zone into smaller pieces by using a regular postgres range partitioning
(sub-tables) on each node.

The main unsolved problems are:

- near neighbor queries. Even though it is possible to slice a large
  table into pieces and distribute across multiple nodes, it is not
  possible to optimize a near neighbor query by taking advantage of data
  locality – GridSQL will still need to do n2 correlations to complete
  the query. In practice a special layer on top of GridSQL is still
  needed to optimize near neighbor queries.

- shared scans.

Another issue is stability, and lack of good documentation.

Also since GridSQL is based on PostgreSQL, it inherits the postgres
“cons”, such as the slow performance (comparing to MySQL) and having to
reload all data every year.

The above reasons greatly reduce the attractiveness of GridSQL.

We have discussed in details the GridSQL architecture with their key
developer, Mason Sharp, who confirmed the issues we identified are
unlikely to be fixed/implemented any time soon.

Gridsql Tests
~~~~~~~~~~~~~

We installed GridSQL on a 3 node cluster at SLAC and run tests aimed to
uncover potential bottlenecks, scalability issues and understand
performance. For these tests we used simulated data generated by the
software built for LSST by the UW team.

Note that GridSQL uses PostgreSQL underneath, so these tests included
installing and testing PostgreSQL as well.

For these tests we used the USNO-B catalog. We run a set of
representative queries, ranging from low volume queries (selecting a
single row for a large catalog, a cone search), to high-volume queries
(such as near-neighbor search).

Our timing tests showed acceptable overheads in performance compared to
PostgreSQL standalone.

We examined all data partitioning options available in GridSQL. After
reading documentation, interacting with GridSQL developers, we
determined that currently in GridSQL, the only way to distribute a table
across multiple nodes is via hash partitioning. We can't simply hash
partition individual objects, as this would totally destroy data
locality, which is essential for spatial joins. A reasonable workaround
we found is to hash partition entire declination zones (hash partition
on zoneId), this will insure all objects for a particular zone end up on
the same node. Further, we can “chunk” each zone into smaller pieces by
using a regular PostgreSQL range partitioning (sub-tables) on each node.

We were unable to find a clean solution for the near neighbor queries.
Even though it is possible to slice a large table into pieces and
distribute across multiple nodes, it is not possible to optimize a near
neighbor query by taking advantage of data locality, so in practice
GridSQL will still need to do n2 correlations to complete the query. In
practice a special layer on top of GridSQL is still needed to optimize
near neighbor queries. So in practice, we are not gaining much (if
anything) by introducing GridSQL into our architecture.

During the tests we uncovered various stability issues, and lack of good
documentation.

In addition, GridSQL is based on PostgreSQL, so it inherits the
PostgreSQL “cons”, such as the slow performance (comparing to MySQL) and
having to reload all data every year, described separately.

.. _infinidb:

InfiniDB
--------

InfiniDB is an open source, columnar DBMS consisting of a MySQL front
end and a columnar storage engine, build and supported by Calpont.
Calpont introduced their system at the MySQL 2008 User Conference
:cite:`Tommaney:2009:MySQLConf`, and more officially `announced it in late Oct 2009
<https://www.prlog.org/10390427-calpont-launches-open-source-analytics-database-offering.html>`_.
It implements true MPP, shared nothing (or shared-all,
depending how it is configured) DBMS. It allows data to be range-based
horizontal partitioning, partitions can be distributed across many nodes
(overlapping partitions are not supported though). It allows to run
*distributed* scans, filter aggregations and hash joins, and offers both
intra- and inter- server parallelism. During cross-server joins: no
direct communication is needed between workers. Instead, they build 2
separate hash maps, distribute smaller one, or if too expensive to
distribute they can put it on the “user” node.

A single-server version of InfiniDB software is available through free
community edition. Multi-node, MPP version of InfiniDB is only available
through commercial, non-free edition, and is closed source.

We are in contact with Jim Tommaney, CTO of the Calpont Corporation
since April 2008. In late 2010 we run the most demanding query – the
near neighbor tests using Calpont. Details of these tests are covered in
:cite:`DMTN-047`.

.. _luciddb:

LucidDB
-------

LucidDB is an open source columnar DBMS. Early startup (version 0.8 as
of March 2009). They have no plans to do shared-nothing (at least there
is no mention of it, and on their main page they mention “great
performance using only a single off-the-shelf Linux or Windows
server.”). Written mostly in java.

.. _dbs-mysql:

MySQL
-----

MySQL utilizes a shared-memory architecture. It is attractive primarily
because it is a well supported, open source database with a large
company (now Oracle) behind it and a big community supporting it. (Note,
however, that much of that community uses it for OLTP purposes that
differ from LSST's needs.) MySQL's optimizer used be below-average,
however it is slowly catching up, especially the MariaDB version.

We have run many, many performance tests with MySQL. These are
documented in :cite:`DMTN-048`.

We are well plugged into the MySQL community, we attended all MySQL User
Conferences in the past 5 years, and talked to many MySQL developers,
including director of architecture (Brian Aker), the founders (Monty
Widenius, David Axmark), and theMySQL optimizer gurus.

There are several notable open-source forks of MySQL:

- The main one, supported by Oracle. After initial period when Oracle
  was pushing most new functionality into commercial version of MySQL
  only [Error: Reference source not found], the company now appears
  fully committed to support MySQL, arguing MySQL is a critical
  component of web companies and it is one of the components of the full
  stack of products they offer. Oracle has doubled the number of MySQL
  engineers and tripled the number of MySQL QA staff over the past year
  :cite:`Ulin:2013:Percona`, and the community seems to believe Oracle is truly committed now
  to support MySQL. The main “problem” from LSST perspective is that
  Oracle is putting all the effort into InnoDB engine only (the engine
  used by web companies including Facebook and Google), while the MyISAM
  engine, the engine of choice for LSST, selected because of vastly
  different access pattern characteristics, remains neglected and Oracle
  currently has no plans to change that.

- MontyProgram and SkySQL used to support two separate forks of MySQL,
  in April 2013 they joint efforts; the two founders of MySQL stand
  behind these two companies. MontyProgram is supporting a viable
  alternative to InnoDB, called MariaDB, and puts lots of efforts into
  improving and optimizing MyISAM. As an example, the mutli-core
  performance issues present in all MySQL engines in the past were fixed
  by Oracle for InnoDB, and in *Aria*, the MontyProgram's version of
  MyISAM by MontyProgram.

- Percona, which focuses on multi-core scaling

- `Drizzle`_, which is a slimmed-down version, rewriten from scratch and no
  longer compatible with MySQL. Based on discussions with the users, the
  `Drizzle`_ effort has not picked up, and is slowly dying.

Spatial indexes / GIS. As of version 5.6.1, MySQL has rewritten spatial
support, added support for spatial indexes (for MyISAM only) and
functions using the OpenGIS geometry model. We have not yet tested this
portion of MySQL, and have preferred using geometry functionality from
SciSQL, a MySQL plug-in written inhouse..

.. _mysql-columnar-engines:

MySQL – Columnar Engines
~~~~~~~~~~~~~~~~~~~~~~~~

.. _kickfire:

KickFire
^^^^^^^^

KickFire is a hardware appliance built for MySQL. It runs a proprietary
database kernel (a columnar data store with aggressive compression) with
operations performed on a custom dataflow SQL chip. An individual box
can handle up to a few terabytes of data. There are several factors that
reduce the attractiveness of this approach:

- it is a proprietary “black box”, which makes it hard to debug, plus it
  locks us into a particular technology

- it is an appliance, and custom hardware tends to get obsolete fairly
  rapidly

- it can handle low numbers of terabytes; another level is still needed
  (on top?) to handle petabytes

- there is no apparent way to extend it (not open source, all-in-one
  “black box”)

We have been in contact with the key people since April of 2007, when
the team gave us a demo of their appliance under an NDA.

.. _infobright:

InfoBright
^^^^^^^^^^

Infobright is a proprietary columnar engine for MySQL. Infobright
Community Edition is open-source, but lacks many features, like
parallelism and DML (INSERT, UPDATE, DELETE, etc). Infobright Enterprise
Edition is closed-source, but supports concurrent queries and DML.
Infobright’s solution emphasizes single-node performance without
discussing distributed operation (except for data ingestion in the
enterprise edition).

.. _sec-tokudb:

TokuDB
~~~~~~

Tokutek built a specialized engine, called TokuDB. The engine relies on
new indexing method, called Fractal Tree indexes :cite:`TokuDB:2013:White`, this new type of
an index primarily increases speed of inserts and data replication.
While its benefits are not obvious for our data access center, rapid
inserts might be useful for Level 1 data sets (Alert Production). We
have been in touch with the Tokutek team for several years, the key
designers of the Fractal Tree index gave a detailed tutorial at the
`XLDB`_-2012 conference we organized.

The engine was made open source in Apr 2013.

.. _netezza:

Netezza
-------

Netezza Performance Server (NPS) is a proprietary, network attached,
*streaming* data warehousing appliance that can run in a shared-nothing
environment. It is built on PostgreSQL.

The architecture of NPS consists of two tiers: a SMP host and hundreds
of massively parallel blades, called Snippet Processing Units (SPU).
Each SPU consists of a CPU, memory, disk drive and an FPGA chip that
filters records as they stream off the disk. See
https://www-01.ibm.com/software/data/netezza/ for more information.

According to some rumours, see e.g. :cite:`Monash:2009:teradata`,
Netezza is planning to support map/reduce.

Pros:

- It is a good, scalable solution

- It has good price/performance ratio.

Cons:

- it is an appliance, and custom hardware tends to get obsolete fairly
  rapidly

- high hardware cost

- proprietary

Purchased by IBM.


.. _oracle:

Oracle
------

Oracle provides scalability through Oracle Real Application Clusters
(RAC). It implements a shared-storage architecture.

Cons: proprietary, expensive. It ties users into specialized (expensive)
hardware (*Oracle Clusterware*) in the form of storage area networks to
provide sufficient disk bandwidth to the cluster nodes; the cluster
nodes themselves are often expensive shared-memory machines as well. It
is very expensive to scale to very large data sets, partly due to the
licensing model. Also, the software is very monolithic, it is therefore
changing very, very slowly.

We have been approached several times by Oracle representatives, however
given we believe Oracle is not a good fit for LSST, we decided not to
invest our time in detailed investigation.

.. _paraccel:

ParAccel
--------

ParAccel Analytic Database is a proprietary RDBMS with a shared-nothing
MPP architecture using columnar data storage. They are big on
extensibility and are planning to support user-defined types, table
functions, user-defined indexes, user-defined operators, user-defined
compression algorithms, parallel stored procedures and more.

When we talked to ParAccel representatives (Rick Glick, in May 2008),
the company was still in startup mode.

.. _postgres:

PostgreSQL
----------

PostgreSQL is an open source RDBMS running in a shared-memory
architecture.

PostgreSQL permits horizontal partitioning of tables. Some large-scale
PostgreSQL-based applications use that feature to scale. It works well
if cross-partition communication is not required.

The largest PostgreSQL setup we are aware of is AOL's 300 TB
installation (as of late 2007). Skype is planning to use PostgreSQL to
scale up to billions of users, by introducing a layer of proxy servers
which will hash SQL requests to an appropriate PostgreSQL database
server, but this is an OLTP usage that supports immense volumes of small
queries :cite:`Hoff:2008:Skype`.

PostgreSQL also offers `good GIS support <http://postgis.refractions.net/>`_ :cite:`Obe:2015:PA:2834495`.
We are collaborating
with the main authors of this extension.

One of the main weaknesses of PostgreSQL is a less-developed support
system. The companies that provide support contracts are less
well-established than Sun/MySQL. Unlike MySQL, but more like Hadoop, the
community is self-organized with no single central organization
representing the whole community, defining the roadmap and providing
long term support. Instead, mailing lists and multiple contributors
(people and organizations) manage the software development.

PostgreSQL is more amenable to modification than MySQL, which may be one
reason why it has been used as the basis for many other products,
including several mentioned below.

Based on the tests we run, PostgreSQL performance is 3.7x worse than
MySQL. We realize the difference is partly due to very different
characteristics of the engines used in these tests (fully ACID-compliant
PostgreSQL vs non-transactional MyISAM), however the non-transactional
solution is perfectly fine, and actually preferred for our immutable
data sets.

We are in touch with few most active PostgreSQL developers, including
the authors of Q3C mentioned above, and Josh Berkus.

.. _postgres-tests:

Tests
~~~~~

We have run various performance tests with PostgreSQL to compare its
performance with MySQL. These tests are described in details in the
“Baseline architecture related” section below. Based on these tests we
determined PostgreSQL is significantly (3.7x) slower than MySQL for most
common LSST queries.

We have also tried various partitioning schemes available in PostgreSQL.
In that respect, we determined PostgreSQL is much more advanced than
MySQL.

Also, during these tests we uncovered that PostgreSQL requires
dump/reload of all tables for each major data release (once per year),
see https://www.postgresql.org/support/versioning. The PostgreSQL
community believes this is unlikely to change in the near future. This
is probably the main show-stopper preventing us from adapting
PostgreSQL.

.. _sec-scidb:

SciDB
-----

SciDB is a new open source system inspired by the needs of LSST15 and
built for scientific analytics. SciDB implements a shared nothing,
columnar MPP array database, user defined functions, overlapping
partitions, and many other features important for LSST. SciDB Release
11.06, the first production release, was published on June 15, 2011. We
are in the process of testing this release.

.. _sqlserver:

SQLServer
---------

Microsoft's SQLServer's architecture is shared-memory. The largest
SQLServer based setup we are aware of is the SDSS database (6 TB), and
the Pan-STARRS database.

In 2008 Microsoft bought DATAllegro and began an effort, codenamed
“Project Madison,” to integrate it into SQLServer. Madison relies on
shared nothing *computing*. Control servers are connected to compute
nodes via dual Infiniband links, and compute servers are connected to
a large SAN via dual Fiber Channel links. Fault tolerance relies on
(expensive) hardware redundancy. For example, servers tend to have
dual power supplies. However, servers are unable to recover from
*storage* node failures, thought a different replica may be used. The
only way to distribute data across nodes is by hashing; the system
relies on replicating *dimension* tables. [the above is based on the
talk we attended :cite:`Dyke:2009:Madison`]

Cons: It is proprietary, relies on expensive hardware (appliance), and
it ties users to the Microsoft OS.

**About DATAllegro**. DATAllegro was a company specializing in data
warehousing server appliances that are pre-configured with a version of
the Ingres database optimized to handle relatively large data sets
(allegedly up to hundreds of terabytes). The optimizations reduce search
space during joins by forcing hash joins. The appliances rely on high
speed interconnect(Infiniband).

.. _sybase-iq:

Sybase IQ
---------

`Sybase IQ`_ is a commercial columnar database product by Sybase Corp.
`Sybase IQ`_ utilizes a “shared-everything” approach that designed to
provide graceful load-balancing. We heard opinions that most of the good
talent has left the company; thus it is unlikely it will be a major
database player.

Cons: proprietary.

.. _teradata:

Teradata
--------

Teradata implements a shared-nothing architecture. The two largest
customers include eBay and WalMart. Ebay is managing multi petabyte
Teradata-based database.

The main disadvantage of Teradata is very high cost.

We are in close contact with Steve Brobst, acting as Teradata CTO, and
key database developers at eBay.

.. _vertica:

Vertica
-------

The Vertica Analytics Platform is a commercial product based on the open
source *C-store* column-oriented database, and now owned by HP. It
utilizes a shared-nothing architecture. Its implementation is quite
innovative, but involves signficant complexity underneath.

It is built for star/snowflake schemas. It currently can not join
multiple fact tables; e.g. self-joins are not supported though this will
be fixed in future releases. Star joins in the MPP environment are made
possible by replicating dimension tables and partitioning the fact
table.

In 2009, a `Vertica Hadoop connector was implemented <https://web.archive.org/web/20091126154136/http://www.vertica.com/MapReduce>`_.
This allows Hadoop
developers to push down map operators to Vertica database, stream Reduce
operations into Vertica, and move data between the two
environments.

Cons:

- lack of support of self-joins

- proprietary..

.. _other-dbs:

Others
------

In addition to map/reduce and RDBMS systems, we also evaluation several
other software packages which could be used as part of our custom
software written on top of MySQL. The components needed include SQL
parser, cluster management and task management.

.. _db-cluster-task-management:

Cluster and task and management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Two primary candidates to use as cluster and task management we
identified so far are Gearman and `XRootD`_. Cluster management involves
keeping track of available nodes, allowing nodes to be added/removed
dynamically. Task management involves executing tasks on the nodes.

Detailed requirements what we need are captured at:
https://dev.lsstcorp.org/trac/wiki/db/Qserv/DistributedFrameworkRequirements

.. _sec-gearman:

Gearman
^^^^^^^

`Gearman`_ is a distributed job execution system, available as open source.
It provides task management functions, e.g., cluster management is left
out to be handled in application code.

During a meeting setup in June 2009 with Eric Day, the key developer
working on integration of `Drizzle`_ with `Gearman`_, who also wrote the C++
version of `Gearman`_, we discussed details of `Gearman`_ architecture and its
applicability for LSST.

`Gearman`_ manages workers as resources that provide RPC execution
capabilities. It is designed to provide scalable access to many
resources that can provide similar functionality (e.g., compress an
image, retrieve a file, perform some expensive computation). While we
could imagine a scheme to use `Gearman`_\ ’s dispatch system, its design did
not match LSST’s needs well. One problem was its store-and-forward
approach to arguments and results, which would mean that the query
service would need to implement its own side transmission channel or
potentially flood the `Gearman`_ coordinator with bulky results.

.. _experiments-on-hive-trac:

Experiments on Hive
===================

This section\ [*]_ discusses an evaluation of the Hive data warehouse
infrastructure for LSST database needs. All experimentation and analysis
done by Bipin Suresh in mid-2010.

Background
----------

`Hive`_ is a data warehouse built upon Hadoop. It defines a SQL-like query
language called QL which allows for queries on structured data. Since it
is built on top of `Hadoop`_, developers can leverage the Map-Reduce
framework to define and perform more complicated analysis by plugging in
their own custom mappers and reducers.

To get Hive running, you need to first have a working version of Hadoop.

Test Conditions
---------------

Software configuration
~~~~~~~~~~~~~~~~~~~~~~

-  Hive 0.7.0
-  Hadoop 0.20.2

Schema
~~~~~~

We have used a reduced Object schema based on USNO-B for testing
purposes. For the final schema that we will be using, please check out
the documentation here:
http://ls.st/8g4

::

    hive> desc object;

+------------+--------+
| id         | int    |
+------------+--------+
| ra         | float  |
+------------+--------+
| decl       | float  |
+------------+--------+
| pm_ra      | int    |
+------------+--------+
| pm_raerr   | int    |
+------------+--------+
| pm_decl    | int    |
+------------+--------+
| pm_declerr | int    |
+------------+--------+
| epoch      | float  |
+------------+--------+
| bmag       | float  |
+------------+--------+
| bmagf      | int    |
+------------+--------+
| rmag       | float  |
+------------+--------+
| rmagf      | int    |
+------------+--------+
| bmag2      | float  |
+------------+--------+
| bmagf2     | int    |
+------------+--------+
| rmag2      | float  |
+------------+--------+
| rmagf2     | int    |
+------------+--------+


Test Queries
~~~~~~~~~~~~

-  Analysis of a single object. Find an object with a particular
   objectId

   .. code:: sql

     hive> select * from object where id=1;

-  Select transient variable objects near a known galaxy

   .. code:: sql

     hive> select v.id, v.ra, v.decl from object v join object o where
     o.id=1 and spDist(v.ra, v.decl, o.ra, o.decl)<10;

-  Analysis of all objects meeting certain criteria

   -  In a region select all galaxies in given area

      .. code:: sql

         hive> select * from object where areaSpec(ra, decl, 0, 0, 10, 10)=true;

   -  For a specified patch of sky, give me the source count density of
      unresolved sources (star like PSF)

      .. code:: sql

         hive> select count(id) from object where areaSpec(ra, decl, 0, 0, 10, 10)=true;

-  Across entire sky. Random sample of the data

   .. code:: sql

      hive> select * from object tablesample(bucket 1 out of 1000 on rand());

-  Analysis of objects close to other objects. Find near-neighbor
   objects in a given region

   .. code:: sql

      hive> select o1.id, o2.id, spDist(o1.ra, o1.decl, o2.ra, o2.decl)
      from object o1 join object o2
      where areaSpec(o1.ra, o1.decl, 0, 0, 10, 10)=true
        and spDist(o1.ra, o1.decl, o2.ra, o2.decl) < 5
        and o1.id <> o2.id;

-  spdist function definition:

   .. code:: java

    package com.example.hive.udf;
    import org.apache.hadoop.hive.ql.exec.UDF;
    import org.apache.hadoop.io.Text;

    public final class SpDist extends UDF {

        public double evaluate(final double ra1, final double dec1, final double ra2, final double dec2) {
            double dra, ddec, a, b, c;
            dra     = radians(0.5 * (ra2 - ra1));
            ddec    = radians(0.5 * (dec2 - dec1));
            a = Math.pow(Math.sin(ddec), 2) + Math.cos(radians(dec1))
                                                      * Math.cos(radians(dec2))
                                                      * Math.pow(Math.sin(dra), 2);
            b = Math.sqrt(a);
            c = b>1 ? 1 : b;
            return degrees(2.0 * Math.asin(c));
        }

        private double radians(double a)    {
            return Math.PI / 180 * a;
        }

        private double degrees(double a)    {
            return a * 180 / Math.PI;
        }


    }

-  areaspec function definition:

   .. code:: java

    package com.example.hive.udf;
    import org.apache.hadoop.hive.ql.exec.UDF;
    import org.apache.hadoop.io.Text;

    public final class AreaSpec extends UDF {

        public boolean evaluate(double ra, double decl, final double raMin, final double declMin,
                                    final double raMax, final double declMax) {
            return (ra > raMin && ra < raMax && decl > declMin && decl < declMax);
        }
    }


Filtering by fields like variability and extendedParameter has been
ignored for now since they are not available in the data. It should be
trivial to add those conditions when the data is ready.

Performance
-----------

Hive is built on top of Hadoop, which is a framework for running
applications across large clusters of commodity hardware. Hadoop/Hive
handles data-distribution and aggregation reliably; and handles
node-failures gracefully. Both data-movement and machine vagaries are
transparent to the user/application-developer.

Single Node
~~~~~~~~~~~

Our first set of experiments were on a single machine, with a local copy
of Hadoop running. The machine was a 64-bit Dual Core AMD Opteron,
running at 1.8GHz, with 4GB of RAM.

On a single node, loading 715K worth of data (~10,000 records with
70bytes each) took 1.178secs. The execution time for each of the queries
are listed below:

-  Q1:

   .. code:: sql

     select * from object where id=1;

   Time taken: 11.88 seconds

-  Q2:

   .. code:: sql

      select v.id, v.ra, v.decl from object v join object o where
         o.id=1 and spDist(v.ra, v.decl, o.ra, o.decl)<10;

   Time taken: 19.999 seconds

-  Q3:

   .. code:: sql

      select * from object where areaSpec(ra, decl, 0, 0, 10, 10)=true;

   Time taken: 10.767 seconds

-  Q4:

   .. code:: sql

      select count(id) from object where areaSpec(ra, decl, 0, 0,10, 10)=true;

   Time taken: 20.77 seconds

-  Q5:

   .. code:: sql

      select * from object tablesample(bucket 1 out of 1000 on rand());

   Time taken: 11.665 seconds

-  Q6:

   .. code:: sql

      select o1.id, o2.id, spDist(o1.ra, o1.decl, o2.ra, o2.decl)
         from object o1 join object o2 where areaSpec(o1.ra, o1.decl, 0,0, 10, 10)=true
            and spDist(o1.ra, o1.decl, o2.ra, o2.decl) < 5
            and o1.id <> o2.id;

   Time taken: 23.053 seconds

Single Node w/Padded Data
~~~~~~~~~~~~~~~~~~~~~~~~~

To simulate the actual data we might be indexing, we padded the schema
with a dummy field 'pad', which ensures that the size of each record >=
1k. The experiments below show the performance of the system with this
dataset. The number of records were kept the same as the above
experiments.

-  Load data time: 1.450s

-  Q1:

   .. code:: sql

      select * from object_padded where id=1;

   Time taken: 12.66 seconds

-  Q2:

   .. code:: sql

      select v.id, v.ra, v.decl from object_padded v join object_padded o where o.id=1
         and spDist(v.ra, v.decl, o.ra,o.decl)<10;


   Time taken: 23.929 seconds

-  Q3:

    .. code:: sql

       select * from object_padded where areaSpec(ra, decl, 0, 0,10, 10)=true;

   Time taken: 10.712 seconds

-  Q4:

   .. code:: sql

      select count(id) from object_padded where areaSpec(ra, decl,0, 0, 10, 10)=true;

   Time taken: 21.81 seconds

-  Q5:

   .. code:: sql

      select * from object_padded tablesample(bucket 1 out of 1000 on rand());

   Time taken: 10.684 seconds

-  Q6:

   .. code:: sql

      select o1.id, o2.id, spDist(o1.ra, o1.decl, o2.ra, o2.decl)
         from object_padded o1 join object_padded o2 where areaSpec(o1.ra, o1.decl, 0, 0, 10, 10)=true
           and spDist(o1.ra, o1.decl, o2.ra, o2.decl) < 5 and o1.id <> o2.id;

   Time taken: 22.837 seconds

Padded Schema
^^^^^^^^^^^^^

::

    hive> desc object_padded;

+------------+--------+
| id         | int    |
+------------+--------+
| ra         | float  |
+------------+--------+
| decl       | float  |
+------------+--------+
| pm_ra      | int    |
+------------+--------+
| pm_raerr   | int    |
+------------+--------+
| pm_decl    | int    |
+------------+--------+
| pm_declerr | int    |
+------------+--------+
| epoch      | float  |
+------------+--------+
| bmag       | float  |
+------------+--------+
| bmagf      | int    |
+------------+--------+
| rmag       | float  |
+------------+--------+
| rmagf      | int    |
+------------+--------+
| bmag2      | float  |
+------------+--------+
| bmagf2     | int    |
+------------+--------+
| rmag2      | float  |
+------------+--------+
| rmagf2     | int    |
+------------+--------+
| pad        | string |
+------------+--------+

3-node cluster
~~~~~~~~~~~~~~

We set up a small 3-node cluster to study the performance of Hive across
multiple machines. The machines were of the same class as the one used
for the single-node experiment. The settings (number of mappers/reducers
etc.) were not tweaked, allowing Hive to determine (guess?) the default
parameters itself. The performance of the cluster is described below:

Load data time: 1.386s

-  Q1:

   .. code:: sql

     select * from object_padded where id=1;

   Time taken: 15.888 seconds

-  Q2:

   .. code:: sql

      select v.id, v.ra, v.decl from object_padded v join
         object_padded o where o.id=1 and spDist(v.ra, v.decl, o.ra, o.decl)<10;

   Time taken: 22.117 seconds

-  Q3:

   .. code:: sql

      select * from object_padded where areaSpec(ra, decl, 0, 0,10, 10)=true;

   Time taken: 12.882 seconds

-  Q4:

   .. code:: sql

      select count(id) from object_padded where areaSpec(ra, decl, 0, 0, 10, 10)=true;

   Time taken: 19.927 seconds

-  Q5:

   .. code:: sql

      select * from object_padded tablesample(bucket 1 out of 1000 on rand());

   Time taken: 10.774 seconds

-  Q6:

   .. code:: sql

     select o1.id, o2.id, spDist(o1.ra, o1.decl, o2.ra, o2.decl)from object_padded o1 join object_padded o2
        where areaSpec(o1.ra,o1.decl, 0, 0, 10, 10)=true
        and spDist(o1.ra, o1.decl, o2.ra, o2.decl) < 5 and o1.id <> o2.id;

   Time taken: 19.996 seconds

At the small data size (10k rows), performance is about the same for
1-node and 3-node.

--------------

Larger scale testing
--------------------

I setup and collected data for different Hive architectures. I tested:

a. a single node setup
b. an 8-node setup
c. a 64-node setup.

Data setup
~~~~~~~~~~

For every architecture, I loaded the 10M rows that Daniel Wang has
generated, and ran the queries we had identified previously

The queries were restricted to queries on the Object table. No padding
was used to increase the size of the rows to 1k.

For each architecture, I ran five runs of each query, and recorded the
average running time. I've attached some analysis to this mail.

Conclusions
~~~~~~~~~~~

The main conclusions from these set of experiments are:

1. Hive scales well with increase in nodes. Increasing the number of
nodes is a configuration change, followed by a restart. I suspect
however, that data might have to re-partitioned/re-indexed if we add
nodes dynamically.

2. Hive scales reasonably well with increase in data-size. A 1,000 fold
increase in data-size (from 10,000 to 10,000,000 records) increased
running times by ~30 times. More experiments will need to be done to pin
that number down.

3. Adding more nodes improves performance: the query execution time
typically drops by 50% when we move from a 1-node setup to an 8-node
setup. Further increases in number of nodes decreases query-execution
time still, but not as drastically. We'll need to perform further
experiments to tease out whether this is because of the (limited)
data-size we're using, or whether it's because of the profile of the
queries.

4. Query-6 stands apart in that it gains almost nothing by the inclusion
of more nodes. Analysis shows that this is primarily because most of the
time of this query is spent in the final Reduce step, which needs to
aggregate the join, and results in a whopping 43k records. Further
experiments will need to be done to identify the root of the bottleneck
- whether it's the large number of results, or whether it's because of
significant (single) reduce step.

+--------------------------------+----------+----------+----------+----------+----------+-----------+
|                                | Q1       | Q2       | Q3       | Q4       | Q5       | Q6        |
+================================+==========+==========+==========+==========+==========+===========+
| 1 node                         |          |          |          |          |          |           |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 1                          | 242.76   | 501.00   | 275.12   | 346.73   | 256.87   | 3868.26   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 2                          | 243.79   | 509.66   | 278.76   | 337.29   | 302.32   | 3969.76   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 3                          | 249.98   | 500.93   | 277.08   | 335.68   | 274.07   | 3828.15   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 4                          | 249.54   | 498.00   | 284.76   | 342.79   | 245.70   | 4027.38   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 5                          | 246.84   | 531.33   | 280.52   | 338.60   | 247.16   | 3629.87   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Average Execution time (sec)   | 246.58   | 508.18   | 279.25   | 340.22   | 265.23   | 3864.68   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| 8 nodes                        |          |          |          |          |          |           |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 1                          | 51.00    | 182.90   | 48.82    | 59.41    | 42.42    | 3470.91   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 2                          | 49.48    | 197.72   | 49.52    | 62.57    | 44.96    | 3546.29   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 3                          | 47.94    | 194.97   | 48.69    | 65.68    | 42.87    | 3438.10   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 4                          | 47.78    | 199.85   | 47.58    | 59.53    | 44.12    | 3495.05   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 5                          | 48.82    | 182.49   | 47.41    | 60.15    | 45.71    | 3482.68   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Average Execution time (sec)   | 49.00    | 191.59   | 48.40    | 61.47    | 44.02    | 3486.61   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| 64 nodes                       |          |          |          |          |          |           |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 1                          | 34.77    | 161.65   | 27.67    | 32.51    | 25.03    | 3499.31   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 2                          | 29.19    | 195.06   | 21.98    | 31.36    | 22.96    | 3722.20   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 3                          | 29.28    | 191.12   | 22.17    | 30.40    | 21.77    | 3582.00   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 4                          | 34.48    | 201.16   | 20.80    | 33.37    | 20.90    | 3473.59   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Run 5                          | 31.94    | 194.98   | 22.17    | 26.31    | 21.92    | 3700.11   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+
| Average Execution time (sec)   | 31.93    | 188.79   | 22.96    | 30.79    | 22.52    | 3595.44   |
+--------------------------------+----------+----------+----------+----------+----------+-----------+

.. figure:: ./_static/HiveScalingPlot.jpg
   :target: ./_static/HiveScalingPlot.jpg
   :alt: Hive scaling plot

Experimentation with Hive was done by Bipin Suresh


.. _community-consult:

People/Communities We Talked To
===============================

Solution providers of considered products:

- Map/Reduce – key developers from Google

- Hadoop – key developers from Yahoo!, founders and key developers
  behind Cloudera and `Hortonworks`_, companyies supporting enterprise
  edition of Hadoop

- `Hive`_ – key developers from Facebook.

- `Dryad`_ – key developers from Microsoft (`Dryad`_ is Microsofts's version
  of map/reduce), including Michael Isard

- `Gearman`_ – key developers (gearman is a system which allows to run
  MySQL in a distributed fashion)

- representatives from all major database vendors, including Teradata,
  Oracle, IBM/DB2, Greenplum, Postgres, MySQL, MonetDB, SciDB

- representatives from promising startups including HadoopDB, ParAcell,
  EnterpriseDB, Calpont, Kickfire

- Intersystem's Cache—Stephen Angevine, Steven McGlothlin

- Metamarkets

User communities:

- Web companies, including Google, Yahoo, eBay, AOL

- Social networks companies, including Facebook, LinkedIn, Twitter,
  Zynga and Quora

- Retail companies, including Amazon, eBay and Sears,

- Drug discovery (Novartis)

- Oil & gas companies (Chevron, Exxon)

- telecom (Nokia, Comcast, ATT)

- science users from HEP (LHC), astronomy (SDSS, Gaia, 2MASS, DES,
  Pan-STARRS, LOFAR), geoscience, biology

Leading database researchers

- M Stonebraker

- D DeWitt

- S Zdonik

- D Maier

- M Kersten


.. _XRootD: http://xrootd.org

.. _XLDB: http://www.xldb.org

.. _CitusDB: https://www.citusdata.com

.. _Hortonworks: https://hortonworks.com/

.. _Hadapt: http://www.teradata.com/products-and-services/Presto/Presto-Download

.. _Hive: https://wiki.apache.org/hadoop/Hive

.. _HBase: http://hbase.apache.org/

.. _Zookeeper: http://zookeeper.sourceforge.net/

.. _Dryad: https://en.wikipedia.org/wiki/Dryad_(programming)

.. _Cassandra: http://cassandra.apache.org/

.. _MongoDB: https://www.mongodb.com/

.. _Drizzle: https://en.wikipedia.org/wiki/Drizzle_(database_server)

.. _SybaseIQ: http://www.sybase.com/products/datawarehousing/sybaseiq

.. _Gearman: http://gearman.org/

.. _Hadoop: http://hadoop.apache.org/

.. _LSST Trac: https://dev.lsstcorp.org/trac/wiki

References
==========

.. bibliography:: bibliography.bib
   :encoding: latex+latin
   :style: lsst_aa

.. [*] A private discussion of this experiment is available at https://listserv.lsstcorp.org/mailman/private/lsst-data/2012-November/310.html.
       The patches required for the MonetDB test can be found at https://github.com/lsst/qserv/tree/tickets/2426

.. [*] Original location of this 2010 report: https://dev.lsstcorp.org/trac/wiki/db/HiveExperiment
