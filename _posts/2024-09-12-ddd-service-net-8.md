---
layout: post
title:  "Domain Driven Design with .NET 8 - Episode 1 - Solution Anatomy"
tags:
-  .NET Core 8
-  Entity Framework Core 8
-  Dapper
-  microservices
-  database
-  MongoDB
-  Clean Architecture
-  CQRS
-  Minimal APIs
-  FastEndpoints
-  GitHub Actions
-  CI/CD
-  Vertical Slicing
author: Maurizio Attanasi
---

This article is the first in a series in which I will annotate concepts related to the architecture, technologies, and organization of a service implemented using the .NET 8 framework.

An example project that demonstrates the concepts we will discuss is available in this repository [https://github.com/maurizioattanasi/ATech.MovieService](https://github.com/maurizioattanasi/ATech.MovieService)

I will begin by describing the anatomy of the solution of a typical service.

<p align='center'>
  <img src='/assets/images/ddd-service-net-8/onion-architecture.jpeg' alt='onion-architecture' style="max-width:100%">
</p>

The image above is one of the possible representations of onion architecture, one of the concepts defined by clean architecture, a software design pattern that offers several advantages, particularly in terms of maintainability, testability, and flexibility.

Each of the layers represented in the project corresponds to a design of the solution.

<p align='center'>
    <img src='/assets/images/ddd-service-net-8/solution.png' alt='solution-layout' style="max-width:70%">
</p>
<br>

| Project Name                              | Description                                                                                                                       | Type     |
| :-------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- | :------- |
| ATech.MovieService.Domain         | The Domain layer encapsulates the heart of the domain models and business logic.                                                         | classlib |
| ATech.MovieService.Application    | The Application layer manages business logic, harnessing the services of the Domain and Infrastructure layers.                    | classlib |
| ATech.MovieService.Infrastructure | Infrastructure manages database access and external services.                                                                     | classlib |
| ATech.MovieService.Api            | Presentation layer that enables interaction with users or external systems, using the services provided by the Application layer. | web      |

<br/>

Enjoy, :wink:
