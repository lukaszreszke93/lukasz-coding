Mocking EF Core In Unit Tests
=============================

Recently I had some issues writing unit tests for service that was heavily using Entity Framework Context.
I decided to mock it, as I was not able to test it with Sqlite or InMemoryDatabase.

## Mocking Query Part
So I started with writing tests for querying part of the service. 
One of the tricky part here is to mock relations properly. This requires some domain knowledge.
Other tricky part was to actually mock DbSets and DbContext to return values.

I ended with method similar to this one to get mocked DbSet:

```csharp
        public Mock<DbSet<Order>> GetOrderDbSet(IList<Order> data)
        {
            var mockSet = new Mock<DbSet<Order>>();

            var queryable = data.AsQueryable();

            mockSet.As<IQueryable<Order>>().Setup(m => m.Provider).Returns(queryable.Provider);
            mockSet.As<IQueryable<Order>>().Setup(m => m.Expression).Returns(queryable.Expression);
            mockSet.As<IQueryable<Order>>().Setup(m => m.ElementType).Returns(queryable.ElementType);
            mockSet.As<IQueryable<Order>>().Setup(m => m.GetEnumerator()).Returns(queryable.GetEnumerator());

            return mockSet;
        }
```

So with that, I was ready to prepare my entity relations and mock up DbSet to return them.

It leaded me to test that was similar to this

```csharp
        [Fact]
        public void Should_return_one_item()
        {
            // Arrange

            var customer = new Customer { CustomerId = "CustomerId" };
            IList<Order> data = new List<Order>
            {
                new Order { OrderId = 1, Customer = customer }
            };

            var orderDbSet = GetOrderDbSet(data);

            var northwindContext = new Mock<NorthwindDbContext>();
            northwindContext.Setup(x => x.Orders).Returns(orderDbSet.Object);

            var sut = new OrdersProvider(northwindContext.Object);

            // Act

            var result = sut.Get();

            // Assert

            result.Count().ShouldBe(1);
        }
```

You get the point.

## Testing Add & SaveChanges
The next functonality that I wanted to test was adding & saving changes.
First I wanted to create functionality where I can add something to my mocked DbSet and read it with query later.
But I was not able to make it work. Maybe I did something wrong (if you know solution, let me know in comment).

So I was forced to do it differently. I extended my mocking methods to be able to track Add and SaveChanges methods of EF Core Context.

```csharp
  public Mock<DbSet<Order>> GetOrderDbSet(IList<Order> data)
        {
            var mockSet = new Mock<DbSet<Order>>();
            var queryable = data.AsQueryable();
            mockSet.As<IQueryable<Order>>().Setup(m => m.Provider).Returns(queryable.Provider);
            mockSet.As<IQueryable<Order>>().Setup(m => m.Expression).Returns(queryable.Expression);
            mockSet.As<IQueryable<Order>>().Setup(m => m.ElementType).Returns(queryable.ElementType);
            mockSet.As<IQueryable<Order>>().Setup(m => m.GetEnumerator()).Returns(queryable.GetEnumerator());

            // Setup Add

            mockSet.Setup(d => d.Add(It.IsAny<Order>())).Callback<Order>((s) => data.Add(s));

            // Setup Update

            mockSet.Setup(d => d.Update(It.IsAny<Order>()));

            // Remove

            mockSet.Setup(d => d.Remove(It.IsAny<Order>())).Callback<Order>((s) => data.Remove(s));


            return mockSet;
        }
```

But as I said, it still wasn't enough to Add something to DbSet and read it.
So I found out that it is possible to check if mocked DbContext method has been called.

I moved my tests into a little bit other direction than I planned in the begining.
Tested method and test looks like this:

```csharp
        public void AddOrderForCustomer(string customerId, Order newOrder)
        {
            if (customerId == null) throw new ArgumentNullException(nameof(customerId));
            if (newOrder == null) throw new ArgumentNullException(nameof(newOrder));

            var customer = _dbContext.Customers.SingleOrDefault(x => x.CustomerId == customerId);

            customer?.Orders.Add(newOrder);

            _dbContext.SaveChanges();
        }
```

```csharp
        [Fact]
        public void SaveChanges_should_be_called()
        {
            // Arrange

            var customer = new Customer { CustomerId = "CustomerId" };

            var newOrder = new Order { OrderId = 7, ShipCity = "Warsaw"};

            IList<Order> data = new List<Order>
            {
                new Order { OrderId = 1, Customer = customer }
            };

            var orderDbSet = GetOrderDbSet(data);
            var customersDbSet = GetCustomerDbSet();

            var northwindContext = new Mock<NorthwindDbContext>();

            northwindContext.Setup(x => x.Orders).Returns(orderDbSet.Object);
            northwindContext.Setup(x => x.Customers).Returns(customersDbSet.Object);

            var sut = new OrdersProvider(northwindContext.Object);

            // Act

            sut.AddOrderForCustomer(customer.CustomerId, newOrder);

            // Assert

            northwindContext.Verify(x => x.SaveChanges(), Times.Once);
        }
```

Notice the line where assertion is done.
The tests still do the job, I am able to check if the entity was updated. That fact was important here. 
And checking it is achieved.
There are other tests to check wether entity did good job with what to update.

## Summary
This is alternative way for testing services/classes that depend on EF Core DbContext.
It provides possibility to verify operations performed on mocked context and also checking wether the query (or result from query) is handled properly.
I find it useful with complex queries, where many different fields can be filtered and joins between tables are applied.
