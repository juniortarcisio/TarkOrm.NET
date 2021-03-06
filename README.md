# TarkOrm

It's a simple Micro ORM system inspired in Dapper, I'm slowly developing some ideas by myself, anyone who'd like to contribute is welcome to share your own ideas, fork or pull a suggestion/fix/implementation/etc. 

My meta is to make something more interesting than Dapper, since I think that some dapper interfaces doesn't sound natural or doesn't looks like a repository.

My meta is also including some automatic basic queries through the entities and some basic extensible repository ideas, let's see where will it take us.

It can be mapped using DataAnnotation.Schema attributes and I hope I'm going to make a basic MappingConfiguration class in the future.

#### I am still expecting to refector it a few some times more before launching an official relase, for now it's only beta or RC.


# Class Diagram

![alt tag](https://raw.githubusercontent.com/juniortarcisio/TarkOrm.NET/master/classDiagram.png)

# Methods

These components are being used in the following usage examples

#### Country Table
```sql
CREATE TABLE [dbo].[Country](
    [CountryID] [int] NOT NULL,
    [CountryCode] [varchar](2) NOT NULL,
    [Name] [varchar](150) NOT NULL,
    [ContinentID] [int] NOT NULL,
    [FlagB64] [varchar](4000) NULL,
    [CurrencyID] [int] NULL,
)
```

#### Country Class
```csharp
public class Country
{
    [Key]        
    public int CountryID { get; set; }
  
    [Column("CountryCode")] /*Optional*/
    public string CountryCode { get; set; }
  
    public string Name { get; set; }
  
    public int ContinentID { get; set; }
  
    public string FlagB64{ get; set; }
  
    public int CurrencyID { get; set; }
}   
```

## Querying a list from a mapped entity

```csharp
public virtual IEnumerable<T> GetAll<T>
```

Example of usage:

```csharp
TarkOrm tarkOrm = new TarkOrm("connectionStringName");
IEnumerable<Country> lista = tarkOrm.GetAll<Country>();
```

Result

![alt tag](https://github.com/juniortarcisio/TarkOrm.NET/blob/master/unitTestGetAll.png?raw=true)


## Selecting an item from a mapped entity

```csharp
public virtual T GetById<T>(params object[] keyValues)
```

Example of usage:

```csharp
TarkOrm tarkOrm = new TarkOrm("connectionStringName");
Country country = tarkOrm.GetById<Country>(10);
```

Result

![alt tag](https://github.com/juniortarcisio/TarkOrm.NET/blob/master/unitTestGetById.png?raw=true)


## Querying on a IDbConnection using extension methods (as Dapper)

```csharp
public static T GetById<T>(this IDbConnection connection, params object[] keyValues)
```

Example of usage:

```csharp
string connectionString = ConfigurationManager.ConnectionStrings["localhost"].ConnectionString;
SqlConnection connection = new SqlConnection(connectionString);
Country country = connection.GetById<Country>(246);
```

## Querying using table hints in a Fluent API (prototype)

It's avaliable on the extension methods for IDbConnection, it allows to use table prefabs table hints or string table hints as indexes and others.

```csharp
public static TarkOrm WithTableHint(this IDbConnection connection, string tableHint)
```

Example of usage:

```csharp
string connectionString = ConfigurationManager.ConnectionStrings["localhost"].ConnectionString;
SqlConnection connection = new SqlConnection(connectionString);
Country country = connection.WithTableHint(TableHints.SQLServer.NOLOCK).GetById<Country>(246);
```

Resulting query

```SQL
SELECT * FROM dbo.Country WITH(NOLOCK) WHERE CountryID = @CountryID 
```

## Inserting an item from a mapped entity

```csharp
public virtual void Add<T>(T entity)
```

Example of usage:

```csharp
TarkOrm tarkOrm = new TarkOrm("connectionStringName");

tarkOrm.Add(new Country
{
    CountryID = 247,
    ContinentID = 1,
    CountryCode = "ND",
    CurrencyID = 1,
    FlagB64 = "",
    Name = "Testing Country"
});
```

Result

![alt tag](https://github.com/juniortarcisio/TarkOrm.NET/blob/master/unitTestInsert.png?raw=true)


## Updating an item from a mapped entity

```csharp
public virtual void Update<T>(T entity)
```


```csharp
TarkOrm tarkOrm = new TarkOrm("localhost");

Country country = tarkOrm.GetById<Country>(247);
country.Name = "Testing Country Update2";
country.ContinentID = 3;
country.CountryCode = "XX";
country.CurrencyID = 3;
country.FlagB64 = "nd";
tarkOrm.Update(country);
```

Result

![alt tag](https://github.com/juniortarcisio/TarkOrm.NET/blob/master/unitTestUpdate.png?raw=true)


## Deleting an item from a mapped entity


```csharp
public virtual void RemoveById<T>(params object[] keyValues)
```

Example of usage:

```csharp
TarkOrm tarkOrm = new TarkOrm("connectionStringName");
tarkOrm.RemoveById<Country>(247);
```


# First benchmarks 

This benchmarks were performed on a I7 4970k, 16gb ram and running SQL Server 2008 on a SSD Disk.

### Over 250 entities

![alt tag](https://raw.githubusercontent.com/juniortarcisio/TarkOrm.NET/master/benchmarkCountry.png)


### Over 50.000 entities

![alt tag](https://raw.githubusercontent.com/juniortarcisio/TarkOrm.NET/master/benchmarkCity.png)

