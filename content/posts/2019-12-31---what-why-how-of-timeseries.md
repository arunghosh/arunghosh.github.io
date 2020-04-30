---
title: What why and how of Time Series database
date: "2019-12-31T22:40:32.169Z"
template: "post"
draft: false
slug: "what-why-and-how-of-timeseries"
category: "Database"
tags:
  - "Time Series"
  - "Database"
description: "An article of what is Time Series data, why do we need Time Series database and how does Time Series database work."
---

If you are following tech, there is good chance to encounter the term *Time Series Database*. This article is indented to convey —

* What is Time Series data?

* Why is Time Series becoming more relevant now?

* Why do we need a separate DB for handling Time Series data?

* How does Time Series DB work?


### What?

As per [Wikipedia](https://en.wikipedia.org/wiki/Time_series)
> A **time series** is a series of data points indexed (or listed or graphed) in time order. Most commonly, a time series is a sequence taken at successive equally spaced points in time.

Time Series data is a sequence of data points. And each data point is a combination of timestamp and a value. For example an application monitoring a server will be sending CPU usage at regular interval. Each entry will be a pair of timestamp and with the CPU usage. And the data will look like —

|Timestamp|Value|
|-|-:|
|2019-12-27T10:50:00|22|
|2019-12-27T10:51:00|24|
|2019-12-27T10:52:00|25|
|2019-12-27T10:53:00|23|
|2019-12-27T10:54:00|23|


It is getting more limelight now because of lot of Time Series data is being generated from sources like—

* IoT sensor — There are sensors every where from from big industries to activity sensor in the wrist watches/bands. They generate large amount of data that need to be stored and processed.

* Stock market price generates large amount of data that need to be processed real-time.

* Server monitoring — Servers need to be monitored for different factors like CPU usage, memory usage, disk IOPS, etc.


## Why?

The above server monitoring data looks simple, right? Simple enough to be handled by most of the data stores like Relational DB(PostgresSQL) or Document Store(MongoDB). Then why do we need a special database to manage time series data? The reason lies in the nature of Time Series data and the way it is managed/processed—

* **High write throughput** — need to support high write through put or rather high rate of INSERT operation. The operation is more of INSERT than update.

* **Analyze large range of data** — the query requirement might be to get the weekly average of the stock value of APPLE for the past 4 years.

* **Data lifecycle management** — there can requirement where in we need to have higher precision for the recent data than old data. So as data becomes it old it need to be aggregated and downsampled.

The above points limits the usage of RDBMS and Document Store from handling Time Series data. To summarize Time Series DB should

* Support high INSERT rate

* Analyze large rage is data

* Manage TS data lifecycle needs

In addition to the above mentioned points some DBs([Prometheus](https://prometheus.io/)) provide monitoring and alerting system.


### How?

A main factor that differentiates Time Series DB and other DBs is Indexing. The Indexing is designed based the fact that Time Series transaction are mostly INSERTs and not UPDATEs.

Different Time Series DBs take different approach for indexing. For example [TimeScaleDB](https://www.timescale.com/) is build on top of [PostgresSQL](https://www.postgresql.org/). It’s index is based on [B-Tree](https://en.wikipedia.org/wiki/B-tree). On the other hand [InfluxDB](https://www.influxdata.com/) has a new index by name [Time Series Index](https://docs.influxdata.com/influxdb/v1.7/concepts/time-series-index/)(TSI) based on [LSM Tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree).

There is an [interesting read](https://blog.timescale.com/blog/time-series-data-why-and-how-to-use-a-relational-database-instead-of-nosql-d0cd6975e87c/) on why TimeScaleDB went for an B-Tree based relational database for Time Series DB, given the fact that LSM Tree supports faster inserts than B-Tree.

### Reference

* [Indexes in PostgreSQL](https://habr.com/en/company/postgrespro/blog/441962/)

* [Time Series Index (TSI) overview](https://docs.influxdata.com/influxdb/v1.7/concepts/time-series-index/)

* [Why (and how) to use a relational database instead of NoSQL](https://blog.timescale.com/blog/time-series-data-why-and-how-to-use-a-relational-database-instead-of-nosql-d0cd6975e87c/)

* [Prometheus](https://prometheus.io/)
