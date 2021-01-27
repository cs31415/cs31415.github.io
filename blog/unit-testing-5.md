---
layout: default
title: Unit Testing Part 5
description: Refactoring Unit Tests
image: refactoring.jpg
---
### {{ page.title }}

![Unit Testing](../../../img/refactoring.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@uniqueton?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Anton</a> on <a href="https://unsplash.com/s/photos/cleaning?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

### Refactoring Unit Tests

Unit tests are code too. They can also get unwieldy and hard to read. Thus, they too can be [refactored](https://martinfowler.com/tags/refactoring.html) using the same principles used for regular code to eliminate clutter and make it easier to read and add new tests.

Let us consider a couple of scenarios:
#### Common setup and teardown code:

Consider our `ActivityDataTests` class from [Unit Testing 4](http://chandrasivaraman.com/blog/unit-testing-4/). We can see that the mock sql client and config objects are being created in every test. 

```csharp
public class ActivityDataTests
{
    [Theory]
    [InlineData(null)]
    [InlineData("")]
    public void Save_InvalidActivity_RaisesException(string activity)
    {
        // Arrange
        var sqlClient = Mock.Of<ISqlClient>();  // <<==
        var config = Mock.Of<IConfig>();        // <<==
        var activityData = new ActivityData(sqlClient, config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save(activity));
    }

    [Fact]
    public void Save_ValidActivity_UpdatesDatabase()
    {
        // Arrange
        var sqlClient = Mock.Of<ISqlClient>();  // <<==
        Mock.Get(sqlClient)
            .Setup(m => m.ExecuteNonQuery(It.IsAny<string>(), It.IsAny<string>()));

        var config = Mock.Of<IConfig>();        // <<==
        Mock.Get(config)
            .SetupGet(p => p.ActivityDbConnectionString)
            .Returns("valid-connection-string");

        var activityData = new ActivityData(sqlClient, config);

        // Act
        activityData.Save("Create a compost pile");

        // Assert
        Mock.Get(sqlClient)
            .Verify(m => m.ExecuteNonQuery(
                It.IsAny<string>(), It.IsAny<string>()), Times.Once);
    }

    [Fact]
    public void Save_InvalidConnectionString_RaisesException()
    {
        // Arrange
        var sqlClient = Mock.Of<ISqlClient>();  // <<== 
        var config = Mock.Of<IConfig>();        // <<==
        string invalidConnectionString = string.Empty;
        Mock.Get(config)
            .SetupGet(p => p.ActivityDbConnectionString)
            .Returns(invalidConnectionString);

        var activityData = new ActivityData(sqlClient, config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save("Create a compost pile"));
    }
}
```

This creation can be moved out into a constructor. xUnit creates a new instance of the test class for each test method so any expectations we setup will be on fresh instances of these objects and there is no potential for interference with other tests. Just by doing this we removed two lines of code per test:

```csharp
public class RefactoredActivityDataTests
{
    private readonly ISqlClient _sqlClient;
    private readonly IConfig _config;

    public RefactoredActivityDataTests()
    {
        _sqlClient = Mock.Of<ISqlClient>();     // <<==
        _config = Mock.Of<IConfig>();           // <<==
    }

    [Theory]
    [InlineData(null)]
    [InlineData("")]
    public void Save_InvalidActivity_RaisesException(string activity)
    {
        // Arrange
        var activityData = new ActivityData(_sqlClient, _config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save(activity));
    }

    [Fact]
    public void Save_ValidActivity_UpdatesDatabase()
    {
        // Arrange
        Mock.Get(_sqlClient)
            .Setup(m => m.ExecuteNonQuery(It.IsAny<string>(), It.IsAny<string>()));

        Mock.Get(_config)
            .SetupGet(p => p.ActivityDbConnectionString)
            .Returns("valid-connection-string");

        var activityData = new ActivityData(_sqlClient, _config);

        // Act
        activityData.Save("Create a compost pile");

        // Assert
        Mock.Get(_sqlClient)
            .Verify(m => m.ExecuteNonQuery(
                It.IsAny<string>(), It.IsAny<string>()), Times.Once);
    }

    [Fact]
    public void Save_InvalidConnectionString_RaisesException()
    {
        // Arrange
        string invalidConnectionString = string.Empty;
        Mock.Get(_config)
            .SetupGet(p => p.ActivityDbConnectionString)
            .Returns(invalidConnectionString);

        var activityData = new ActivityData(_sqlClient, _config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save("Create a compost pile"));
    }
}
```

#### Setting up expectations on mock objects:

Setting up expectations on mock objects is also repetitive code that can get messy and hinder readability, especially when we have multiple dependencies and long parameter lists. In this case, we could extract out this code into a mock wrapper class:

```csharp
public class MockSqlClient : Mock<ISqlClient>
{
    public void Setup_ExecuteNonQuery()
    {
        this.Setup(m => m.ExecuteNonQuery(It.IsAny<string>(), It.IsAny<string>()));
    }

    public void Verify_ExecuteNonQuery_Called(Func<Times> times)
    {
        this.Verify(m => m.ExecuteNonQuery(
            It.IsAny<string>(), It.IsAny<string>()), times);
    }
}
```

We could use it like this (note that we have to reference the `.Object` property on the mock object to get the type being mocked):

```csharp
public class RefactoredActivityDataTests
{
    private readonly MockSqlClient _mockSqlClient;
    private readonly IConfig _config;
  
    public RefactoredActivityDataTests()
    {
        _mockSqlClient = new MockSqlClient();
        _config = Mock.Of<IConfig>();
    }

    [Theory]
    [InlineData(null)]
    [InlineData("")]
    public void Save_InvalidActivity_RaisesException(string activity)
    {
        // Arrange
        var activityData = new ActivityData(_mockSqlClient.Object, _config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save(activity));
    }

    [Fact]
    public void Save_ValidActivity_UpdatesDatabase()
    {
        // Arrange
        _mockSqlClient.Setup_ExecuteNonQuery(); // <<==

        Mock.Get(_config)
            .SetupGet(p => p.ActivityDbConnectionString)
            .Returns("valid-connection-string");

        var activityData = new ActivityData(_mockSqlClient.Object, _config);

        // Act
        activityData.Save("Create a compost pile");

        // Assert
        _mockSqlClient.Verify_ExecuteNonQuery_Called(Times.Once); // <<==
    }

    [Fact]
    public void Save_InvalidConnectionString_RaisesException()
    {
        // Arrange
        string invalidConnectionString = string.Empty;
        Mock.Get(_config)
            .SetupGet(p => p.ActivityDbConnectionString)
            .Returns(invalidConnectionString);

        var activityData = new ActivityData(_mockSqlClient.Object, _config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save("Create a compost pile"));
    }
}
```

Doing the same for the config object yields this more concise and readable version of the test class:
```csharp
public class RefactoredActivityDataTests
{
    private readonly MockSqlClient _mockSqlClient;
    private readonly MockConfig _mockConfig;

    public RefactoredActivityDataTests()
    {
        _mockSqlClient = new MockSqlClient();
        _mockConfig = new MockConfig();
    }

    [Theory]
    [InlineData(null)]
    [InlineData("")]
    public void Save_InvalidActivity_RaisesException(string activity)
    {
        // Arrange
        var activityData = 
            new ActivityData(_mockSqlClient.Object, _mockConfig.Object);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save(activity));
    }

    [Fact]
    public void Save_ValidActivity_UpdatesDatabase()
    {
        // Arrange
        _mockSqlClient.Setup_ExecuteNonQuery();
        _mockConfig.SetupGet_ActivityDbConnectionString("valid-connection-string");

        var activityData = 
            new ActivityData(_mockSqlClient.Object, _mockConfig.Object);

        // Act
        activityData.Save("Create a compost pile");

        // Assert
        _mockSqlClient.Verify_ExecuteNonQuery_Called(Times.Once);
    }

    [Fact]
    public void Save_InvalidConnectionString_RaisesException()
    {
        // Arrange
        _mockConfig.SetupGet_ActivityDbConnectionString(string.Empty);

        var activityData = 
            new ActivityData(_mockSqlClient.Object, _mockConfig.Object);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save("Create a compost pile"));
    }
}
```

### Code
[https://github.com/cs31415/unit-test-samples](https://github.com/cs31415/unit-test-samples)

### Takeaways:
- Unit tests can be refactored to make them more readable and maintainable
- Consolidating common setup code in a constructor and setting up a mock wrapper are two techniques for refactoring unit tests
