---
layout: post
title:  "Enhancing the Repository Pattern with Id Generalization"
tags:
-  .NET Core 8
-  Entity Framework Core 8
-  Dapper
-  microservices
-  database
-  SQL server
-  SQLite
-  GitHub Actions
-  CI/CD
author: Maurizio Attanasi
---

Our previous article [Repository Pattern, Unit Of Work with EF Core and Dapper](https://maurizioattanasi.it/2021/05/31/repository-pattern.html) discussed the Repository Pattern and its implementation using Entity Framework Core and Dapper. In this article, we introduce an enhanced version of the pattern that increases flexibility and robustness in your data access layer.

## The challenge

In the original Repository Pattern, we often dealt with specific entity types and their corresponding primary keys - usually named 'Id'. However, maintaining separate repositories for each entity type becomes cumbersome as the application grows. How can we make the handling of primary keys more general?

## The Solution

We introduce a new concept - the generic Id - which allows us to handle various entity types seamlessly. Instead of relying on a specific type for the primary key, we use a generic type parameter.

```csharp
public interface IRepository<TEntity, TId>
{
    TEntity Get(TId id);
    ValueTask<TEntity> GetAsync(TId id, CancellationToken cancellationToken);
    ...
```

and it's implementations:

- for Entity Framework

```csharp
    public virtual TEntity Get(TId id) 
        => _context.Set<TEntity>().Find(id);

    public virtual async ValueTask<TEntity> GetAsync(TId id, CancellationToken cancellationToken) 
        => await _context.Set<TEntity>().FindAsync(new object[] { id }, cancellationToken);
```

- and for Dapper

```csharp
    public TEntity Get(TId id)
        => _connection.Get<TEntity, TId>(id);

    public async ValueTask<TEntity> GetAsync(TId id, CancellationToken cancellationToken)
        => await _connection.GetAsync<TEntity, TId>(id, cancellationToken);
```

## Code and NuGet packages

You can find the code on my GitHub profile by clicking on the following link: [ATech.Repository](https://github.com/maurizioattanasi/ATech.Repository). The repository also includes a continuous integration and continuous deployment workflow, which is implemented using [GitHub Actions](https://github.com/features/actions). As a result of this workflow, three NuGet packages are created and published, which are available at [https://www.nuget.org/packages?q=ATech.Repository](https://www.nuget.org/packages?q=ATech.Repository).

<br/>

Enjoy, :wink:
