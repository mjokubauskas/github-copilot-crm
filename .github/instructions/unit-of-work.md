# Unit of Work with Dapper

## Overview

The Unit of Work pattern is a design pattern that allows you to group multiple operations into a single transaction. This is particularly useful for managing complex business transactions where multiple changes need to be made to the database.

Dapper is a lightweight ORM for .NET, which makes it easy to interact with the database. Below are the steps on how to implement the Unit of Work pattern with Dapper.

## Implementation Steps

### 1. Create a Unit of Work Interface
Define an interface for the Unit of Work, specifying methods for handling commits and repositories.

```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<T> GetRepository<T>() where T : class;
    void Commit();
}
```

### 2. Implement the Unit of Work
Create a class that implements the `IUnitOfWork` interface.

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly IDbConnection _connection;
    private IDbTransaction _transaction;

    public UnitOfWork(IDbConnection connection)
    {
        _connection = connection;
        _connection.Open();
        _transaction = _connection.BeginTransaction();
    }

    public IRepository<T> GetRepository<T>() where T : class
    {
        return new Repository<T>(_connection, _transaction);
    }

    public void Commit()
    {
        _transaction.Commit();
    }

    public void Dispose()
    {
        _transaction?.Dispose();
        _connection?.Dispose();
    }
}
```

### 3. Create a Repository Interface
Define a repository interface to outline standard data operations.

```csharp
public interface IRepository<T> where T : class
{
    void Add(T entity);
    void Update(T entity);
    void Delete(T entity);
    T GetById(int id);
    IEnumerable<T> GetAll();
}
```

### 4. Implement the Repository
Create a class for the repository, implementing CRUD operations using Dapper.

```csharp
public class Repository<T> : IRepository<T> where T : class
{
    private readonly IDbConnection _connection;
    private readonly IDbTransaction _transaction;

    public Repository(IDbConnection connection, IDbTransaction transaction)
    {
        _connection = connection;
        _transaction = transaction;
    }

    public void Add(T entity)
    {
        // Implement add logic using Dapper
    }

    public void Update(T entity)
    {
        // Implement update logic using Dapper
    }

    public void Delete(T entity)
    {
        // Implement delete logic using Dapper
    }

    public T GetById(int id)
    {
        // Implement get by id logic using Dapper
    }

    public IEnumerable<T> GetAll()
    {
        // Implement get all logic using Dapper
    }
}
```

### 5. Using the Unit of Work
You can now use the Unit of Work pattern in your application as follows:

```csharp
using (var unitOfWork = new UnitOfWork(new SqlConnection(connectionString)))
{
    var repository = unitOfWork.GetRepository<YourEntity>();
    // Perform operations
    unitOfWork.Commit();
}
```

## Conclusion
Implementing the Unit of Work pattern with Dapper allows for better management of database transactions and keeps your code organized. Adjust the interface and implementation according to your project requirements.