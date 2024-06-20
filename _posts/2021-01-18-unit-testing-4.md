---
layout: post
category: blog
title: Unit Testing Part 4 
permalink: /blog/unit-testing-4
description: Mocking - Behavior Verification
image: behavior-verification.jpg
---

![Unit Testing](../../../img/behavior-verification.jpg)

### Mocking - Behavior Verification

So, in [part 3](/blog/unit-testing-3), we saw how to mock dependencies. Until now, we have done only state verification. i.e. we run the test and check the state of the System Under Test (SUT) after the test to verify if the test passed. Now, we will take a look at how to do behavior verification (see [part 1](/blog/unit-testing-1) for a discussion of state vs behavior verification). In other words, how to setup an expectation on SUT dependencies such as SQL or API client.

Here's a data access class that saves an activity (see [part 3](/blog/unit-testing-3)) to a SQL database. It takes in a SQL client and a config object.
The `Save` method validates the activity and connection string before executing the SQL update or insert.

```csharp
public class ActivityData
{
    private readonly ISqlClient _sqlClient;
    private readonly IConfig _config;

    public ActivityData(ISqlClient sqlClient, IConfig config)
    {
        _sqlClient = sqlClient;
        _config = config;
    }

    public void Save(string activity)
    {
        if (string.IsNullOrEmpty(activity))
        {
            throw new Exception("Can't save null or empty activity");
        }
        if (string.IsNullOrEmpty(_config.ActivityDbConnectionString))
        {
            throw new Exception("Invalid connection string passed");
        }

        // This is just for illustration purposes. This should be a parameterized 
        // query to prevent SQL injection attacks.
        var cmdText = $"exec sp_SaveActivity '{activity}'";
        _sqlClient.ExecuteNonQuery(_config.ActivityDbConnectionString, cmdText);
    }
}
```

Since the method returns `void`, there is no state that we can verify after invoking the SUT. Behavior verification to the rescue.
Here's a test that passes in an invalid activity value (note that we are using a `theory` test here - see [part 2](/blog/unit-testing-2)).
This is a parameterized test that runs for activity values `null` and `""` (empty string) specified in the `InlineData` attribute.
It validates that the SUT raises an exception when an invalid activity is passed.

```csharp
public class ActivityDataTests
{
    [Theory]
    [InlineData(null)]
    [InlineData("")]
    public void Save_InvalidActivity_RaisesException(string activity)
    {
        // Arrange
        var sqlClient = Mock.Of<ISqlClient>();
        var config = Mock.Of<IConfig>();
        var activityData = new ActivityData(sqlClient, config);

        // Act, Assert
        Assert.Throws<Exception>(() => activityData.Save(activity));
    }
}
```

In the same vein, we can also verify that an invalid connection string results in an exception:
```csharp
[Fact]
public void Save_InvalidConnectionString_RaisesException()
{
    // Arrange
    var sqlClient = Mock.Of<ISqlClient>();
    var config = Mock.Of<IConfig>();
    string invalidConnectionString = string.Empty;
    Mock.Get(config)
        .SetupGet(p => p.ActivityDbConnectionString)
        .Returns(invalidConnectionString);

    var activityData = new ActivityData(sqlClient, config);

    // Act, Assert
    Assert.Throws<Exception>(() => activityData.Save("Create a compost pile"));
}
```

Now, if a valid activity value is passed, we would expect that the SQL client `ExecuteNonQuery` method is called exactly once. We use the `Mock.Verify` method to set this up. This method takes in an expression (`m => m.ExecuteNonQuery(It.IsAny<string>(), It.IsAny<string>())`) and a `Times` parameter specifying how many times the method was invoked:

```csharp
[Fact]
public void Save_ValidActivity_UpdatesDatabase()
{
    // Arrange
    var sqlClient = Mock.Of<ISqlClient>();
    Mock.Get(sqlClient)
        .Setup(m => m.ExecuteNonQuery(It.IsAny<string>(), It.IsAny<string>()));

    var config = Mock.Of<IConfig>();
    Mock.Get(config)
        .SetupGet(p => p.ActivityDbConnectionString)
        .Returns("valid-connection-string");
    
    var activityData = new ActivityData(sqlClient, config);

    // Act

    activityData.Save("Create a compost pile");

    // Assert

    // Here we verify that the ExecuteNonQuery method was called exactly once
    Mock.Get(sqlClient)
        .Verify(m => m.ExecuteNonQuery(
            It.IsAny<string>(), It.IsAny<string>()), Times.Once);
}
``` 

#### Code
[https://github.com/cs31415/unit-test-samples](https://github.com/cs31415/unit-test-samples)

#### Takeaways
- [_Moq_](https://github.com/Moq/moq4/wiki/Quickstart) can be used for behavior verification in addition to state verification.
- Behavior verification lets us setup and verify expectations on mock objects as opposed to verifying SUT state.
- This is helpful when it is tedious to setup mock responses or when the SUT method doesn't return any value to inspect.

Continue to [part 5](/blog/unit-testing-5).