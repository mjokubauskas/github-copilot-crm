# Repository Pattern

The Repository Pattern is a design pattern used to abstract the data layer, allowing for flexible data access while encapsulating the data access logic. In this document, we will discuss how to set up a Repository Pattern using Dapper.

## What is Dapper?

Dapper is a simple object mapper for .NET, which greatly speeds up the process of interacting with the database. It provides a framework for executing SQL queries and mapping results to objects with minimal overhead.

## Setting Up the Repository Pattern using Dapper

1. **Create the Repository Interface**:
   - Define a repository interface that describes the operations for accessing the data.
   ```csharp
   public interface IRepository<T>
   {
       IEnumerable<T> GetAll();
       T GetById(int id);
       void Add(T entity);
       void Update(T entity);
       void Delete(int id);
   }
   ```

2. **Implement the Repository**:
   - Create a concrete class that implements the repository interface. Use Dapper to execute SQL queries.
   ```csharp
   public class Repository<T> : IRepository<T>
   {
       private readonly IDbConnection _dbConnection;

       public Repository(IDbConnection dbConnection)
       {
           _dbConnection = dbConnection;
       }

       public IEnumerable<T> GetAll()
       {
           return _dbConnection.Query<T>("SELECT * FROM " + typeof(T).Name);
       }

       public T GetById(int id)
       {
           return _dbConnection.QuerySingleOrDefault<T>("SELECT * FROM " + typeof(T).Name + " WHERE Id = @Id", new { Id = id });
       }

       public void Add(T entity)
       {
           _dbConnection.Execute("INSERT INTO " + typeof(T).Name + " ...", entity);
       }

       public void Update(T entity)
       {
           _dbConnection.Execute("UPDATE " + typeof(T).Name + " SET ... WHERE Id = @Id", entity);
       }

       public void Delete(int id)
       {
           _dbConnection.Execute("DELETE FROM " + typeof(T).Name + " WHERE Id = @Id", new { Id = id });
       }
   }
   ```

3. **Usage**:
   - Now you can simply instantiate your repository and use it in your application.
   ```csharp
   var repo = new Repository<YourEntityType>(yourDbConnection);
   var allEntities = repo.GetAll();
   ```

## Conclusion

Using the Repository Pattern with Dapper helps keep your data access operations organized and maintainable, promoting a clean separation of concerns.