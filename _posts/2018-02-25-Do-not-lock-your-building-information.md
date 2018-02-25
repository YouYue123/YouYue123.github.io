---
layout: post
section-type: post
title: Do not lock in your BIM data
category: Category
tags: [ 'airsquire', 'data' ]
---

## What is BIM

Building information modeling (BIM) is a process involving the generation and management of digital representations of physical and functional characteristics of places. 

### The good side

![image](/img/bim-adoption.png)

Building information modeling is [rasing dramaticlly](https://www.thenbs.com/knowledge/nbs-national-bim-report-2017) to make smarter decision during the whole life cycle for a building 

**plan --> design --> construct --> operate --> maintain**

BIM is adopted by lots of the countries' government as an standard industry practice. From the [report from GeoSpatial World](https://www.geospatialworld.net/blogs/bim-adoption-around-the-world/), take Germany for example, in 2015 the government announced the formation of the Digital Building Platform — a BIM task group created by several industry-led organizations to develop a national BIM strategy.

### The bad side

![image](/img/bim-market-share.jpg)

But in practice, there is no standard way to build BIM. For example, structure architect will use TEKLA structures for steel structure design. BIM engineer will consolidate a detailed model in Autodesk Revit. Project manager will use VICO Office to do a 4D scheduling. 

This may even more complex. Because there are typically more than 1 contractors for a building development and most of them are not sharing the same software, there are huge interoperability issues. And even for same software, [upgrading version will cause a huge compatibility concern](https://forums.autodesk.com/t5/autocad-forum/autocad-2018-backwards-compatibility/td-p/6998902). It is destroying the core meaning for BIM -- exchangable for all people in project to get a clear information.

## OpenBIM Standard

In order to tackle this information sharing problem, the best way is to construct a standard like the web world to make agreement across different vendors.

![image](/img/open-bim.png)

### Effort from major vendors

In 2016 Autodesk and Trimble issued a [joint press release](https://www.businesswire.com/news/home/20160614005404/en/) announcing an agreement to increase interoperability for customers to gain flexibility throughout the BIM project lifecycle. But other vendors like Bentley, [they do not agree with the current approach to achieve openBIM](http://www.bimplus.co.uk/people/what-bi4m-means-soft4ware-prov8ider/).

### Effort from other organizations

There are lots of organizations like [buildingSMART](https://www.buildingsmart.org/), [openBIM](http://www.openbim.org/) who wants to make BIM more open, standard and solid.  And IFC is invented and are designed as a standard for sharing 3D model data and other related data. IFC stands for Industry Foundation Classes. The Industry Foundation Classes (IFC) data model is intended to describe building and construction industry data. It is a platform neutral, open file format specification that is not controlled by a single vendor or group of vendors.

### Ceveat

As you can see, the IFC format is designed by third-party organization but in-practice software is developed by other vendors. There is a gap to adopt IFC in for existing software companies. And from some of our customers' feedback, there is always a data loss when sharing data between different vendors using IFC.

## How Airsquire do it 

## Trimble Connect + Autodesk Forge  

![image](/img/trimble-connect.png)

![image](/img/forge-logo.png)

In Airsquire, we are practicing SaaS standard for the data. The result of AI verifier behind AirSync will always be shown as open standard format. 

Furthermore, we are currently integrating with Trimble Connect to make **everyone and everything** of your BIM be connected. By using it, customer can easily upload verified BIM model to cloud space and import it **anywhere** such as Revit, Tekla structure, sketchup and so on.

For data input, we are engaging with Autodesk Forge platform to convert design file into open standard formats. By integrating with this, Airsync will support more than 60 formats and BIM from any Autodesk software can be directly understood by our AI verifier.

## Ceveat 

Because of the limitation of [Model Derivative API](https://developer.autodesk.com/api/model-derivative-and-viewer-cover-page/). AirSync will only natively support BIM from Autotesk software. But customer can provide IFC data if it is supported from their BIM software. And in the meantime we are also working to provide API to support non-Autodesk format native supporting for our major customers.