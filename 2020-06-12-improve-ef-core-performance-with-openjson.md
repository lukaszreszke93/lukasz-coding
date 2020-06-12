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
