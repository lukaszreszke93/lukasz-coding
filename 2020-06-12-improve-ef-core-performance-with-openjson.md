# Speed Up Your EF Core Query Performance Using This Simple SQL Trick

If you ever had problem with your query timing out due to filtering by too many ids, you've came to good place.
You can solve it by using openjson filtering in your EF Core query.

This article presents an alternate way for standard filtering with `Where` linq extension.
The result is query that is executed quicker. And the most important, it doesn't time out due too many filter parameters.

## What is `openjson`

In T-SQL there's a mechanism called openjson.

OpenJson is a table-valued function that parses JSON text and returns objects nad properties from the JSON input as rows and columns.
It provides a rowset view over a JSON document.

OpenJson returns a set of rows. Due to this fact, you can use it in FROM clause of Transact SQL statement.

_Note that OpenJson function is only available for databases that have compatibility level 130 or higher._

### Example of a query used to filter collection using openjson

```SQL
    DECLARE @employeeIds NVARCHAR(4000) = N'[1,2,3,4]'

    SELECT * FROM employee
    INNER JOIN OPENJSON(@employeeIds) selectedEmployee
        ON employee.Id = selectedEmployee.value

```

This example provides a list of identifiers as a JSON array of numbers. The query converts the JSON array to a table of identifiers and filters all employees with ID specified within the list.

This query is a equivalent to

```SQL
    SELECT * FROM employee
    WHERE employee.Id IN (1,2,3,4)
```

The difference is that in the second query the parameters are embed within the query.

_This is important difference which we'll use later on in EF Core_

## Implementation of openjson in EF Core

In our example let's assume that the collection will be filtered by ids.
In the sample codes I used EF Core 3.1 .

First, we'll create FilterItem class which will be used in EF Context and in the query.

```c#
    public class FilterItem
    {
        public int Value { get; set; }
    }
```

Second, we have to create property for openjson filtering in Context.

```c#
    public virtual IQueryable<FilterItem> OpenJsonFilter(string ids) =>
        Set<FilterItem>().FromSqlInterpolated($"select value from openjson( {ids} ");
```

In the model builder we have to specify that FilterItem has no key.
This helps EF Core interpreter to use this property correctly.

```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<FilterItem>().HasNoKey();
    }
```

That is actually all the setup that we need.

### Usage of OpenJsonFilter in query

OpenJson filtering is slightly different than regular filtering. As shown in T-SQL example, the filtering will not be just using `.Where()` extension. We'll have to use `Join()` instead.

For the purpose of the example, let's say we have to find all orders for selected customers.

```c#
    public class OrderQuery
    {
        public IEnumerable<int> CustomerIds {get; set;}
        // Other properties for querying against customers
    }

    public class OrderProvider
    {
        private Context _context; // injected via ctor

        public Order GetByCustomerIds(OrderQuery query)
        {
            var ids = JsonSerializer.Serialize(query.CustomerIds);

            var result = _context.Orders
                .Join(
                    _context.OpenJsonFilter(ids),
                    order => order.Id,
                    jsonFilter => jsonFilter.Value,
                    (order. jsonFilter) => order
                );

            return result;
        }
    }
```
