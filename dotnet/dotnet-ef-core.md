# EF Core

## Installation

To begin with, you'll need a package reference to `Microsoft.EntityFrameworkCore.*` where the `*` is the database of your choice, such as `SqlServer`.

For postgres, you'll need to reference `Npgsql.EntityFrameworkCore.PostgreSQL`.

It's a common mistake to both include the database provider (or driver) package and the core `Microsoft.EntityFrameworkCore` package. It's enough to just reference the provider.

## Migrations

Migrations are hugely helpful to keep your C# models in sycn with your database objects, such as tables.

You need a reference to `Microsoft.EntityFramework.Design`. This enables us to use dotnet commands like `dotnet ef