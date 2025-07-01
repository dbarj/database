# Latency Test

## Introduction

In Oracle Autonomous Database on shared infrastructure, it is a common requirement for end customers and internal operations to quickly determine the latency while connecting to a particular service.

A typical use case for such a test would be when a customer query or a workload is running with an unusually high latency and the user needs to ensure if the database service used by the workload, is healthy and is not having inherent latency issues impacting the workload / queries.

*adbping* is an easy to use command line tool that can easily help end users determine the connection and SQL execution latency to benchmark the performance outside of their business workload.

Tool is feature rich, allowing users to run the benchmark test with multitude of options including multi client support, multi threaded, configurable connection characteristics like pool size and other options.

Estimated Time: 20 Minutes

### Objectives

In this lab, you will:

* Use *adbping* to check and compare ADB latency.

### Prerequisites

This lab assumes:

- You have completed Lab 3: Getting Started

## Task 1: SecureFile LOBs

Oracle Database stores LOBs in two formats. *SecureFile LOBs* is the newest format with a lot of advantages over the older format, *BasicFile LOBs*. Oracle recommends that you store your LOBs in SecureFile format.

 

You may now *proceed to the next lab*.

## Additional information

* Webinar, [Data Pump Best Practices and Real World Scenarios, LOB data and Data Pump and things to know](https://www.youtube.com/watch?v=960ToLE-ZE8&t=1798s)
* Webinar, [Data Pump Best Practices and Real World Scenarios, Statistics and Data Pump](https://www.youtube.com/watch?v=960ToLE-ZE8&t=1117s)
* [Speeding up Database Constraints](https://www.youtube.com/watch?v=lgFc0cduPJk&t=299s)

## Acknowledgments

* **Author** - Rodrigo Jorge
* **Contributors** - William Beauregard, Daniel Overby Hansen, Mike Dietrich, Klaus Gronau, Alex Zaballa
* **Last Updated By/Date** - Rodrigo Jorge, May 2025