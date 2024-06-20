---
layout: post
category: blog
title: Unit Testing Part 3 
permalink: /blog/unit-testing-3
description: Mocking dependencies in Unit Tests
image: mockingbird.jpg
---

![Unit Testing](../../../img/mockingbird.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@jcotten?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Joshua J. Cotten</a> on <a href="https://unsplash.com/s/photos/mockingbird?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

### Mocking dependencies in Unit Tests

So, in [part 1](/blog/unit-testing-1), we had an overview of unit tests and mocking and in [part 2](/blog/unit-testing-2), we saw a couple of simple tests. In this post, we'll look at some examples of mocking dependencies. Consider the C# code below:

```csharp
using System.Net.Http;
using Newtonsoft.Json.Linq;

namespace UnitTestSamples
{
    public class ActivityGenerator
    {
        public string GetActivity()
        {
            using (var webClient = new HttpClient())
            {
                var uri = "https://www.boredapi.com/api/activity";
                var responseJson = webClient.GetStringAsync(uri).Result;
                if (!string.IsNullOrEmpty(responseJson))
                {
                    dynamic jObj = JObject.Parse(responseJson);
                    var activity = jObj.activity?.ToString();
                    return activity;
                }
            }

            return null;
        }
    }
}
```

The class `ActivityGenerator` has a single public method `GetActivity` that calls an external API to get a random activity ("Pull a harmless prank on one of your friends", "Create a compost pile" etc.) to add to our already overflowing todo lists. The JSON response looks like:
```json
{
    "activity": "Make a new friend",
    "type": "social",
    "participants": 1,
    "price": 0,
    "link": "",
    "key": "1000000",
    "accessibility": 0
}
```

The `GetActivity` method returns the `activity` property from this JSON.

To write a unit test for this method, we would create a test project, add a test class and add a test method, like this:
```csharp
using Xunit;

namespace UnitTestSamples.Tests
{
    public class ActivityGeneratorTests
    {
        [Fact]
        public void GetActivity_ValidResponse_ReturnsActivity()
        {
            // Arrange
            var activityGenerator = new ActivityGenerator();

            // Act
            var activity = activityGenerator.GetActivity();

            // Assert
            Assert.True(!string.IsNullOrEmpty(activity));
        }
    }
}
```

As mentioned in [part 2](/blog/unit-testing-2), we use the [ASR](https://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html) and [AAA](http://wiki.c2.com/?ArrangeActAssert) patterns used for naming the test method and structuring the test code. 

If you run this test, it will return success. However, it would have made a live call to the API. This is not what we want. What if the API is temporarily down, or is still under development, or we want to test for a failure condition? Calling the live API doesn't help us in any of these scenarios. To add insult to injury, if we have lots of these kind of tests, we'll be spending significant time waiting for tests to finish.   

We can do better by using mocks (See [part 1](/blog/unit-testing-1)). We'll need to add a Nuget reference to [Moq](https://www.nuget.org/packages/moq/), a popular mocking framework. To mock or simulate the web API call, we'll need to "inject" the `HttpClient` instance into the `ActivityGenerator` instead of having it created inside the class. **Moq** works by creating alternative implementations of interfaces. The built-in `HttpClient` class doesn't have an interface we can mock. So, we'll wrap it in an `ApiClient` class and pass its interface into `ActivityGenerator`.

ActivityGenerator.cs:
```csharp
using Newtonsoft.Json.Linq;

namespace UnitTestSamples
{
    public class ActivityGenerator
    {
        private readonly IApiClient _apiClient;

        public ActivityGenerator(IApiClient apiClient)
        {
            _apiClient = apiClient;
        }
        public string GetActivity()
        {
            var uri = "https://www.boredapi.com/api/activity";
            var response = _apiClient.GetAsync(uri).Result;
            var responseJson = response.Content.ReadAsStringAsync().Result;
            if (!string.IsNullOrEmpty(responseJson))
            {
                dynamic jObj = JObject.Parse(responseJson);
                var activity = jObj.activity?.ToString();
                return activity;
            }

            return null;
        }
    }
}

```

ApiClient.cs:
```csharp
using System.Net.Http;
using System.Threading.Tasks;

namespace UnitTestSamples
{
    public class ApiClient : IApiClient
    {
        readonly HttpClient _httpClient;

        public ApiClient()
        {
            _httpClient = new HttpClient();
        }

        public async Task<HttpResponseMessage> GetAsync(string url)
        {
            return await _httpClient.GetAsync(url);
        }
    }
}
```

IApiClient.cs:
```csharp
using System.Net.Http;
using System.Threading.Tasks;

namespace UnitTestSamples
{
    public interface IApiClient
    {
        Task<HttpResponseMessage> GetAsync(string url);
    }
}
```

Now, we can rewrite the test to inject the mock:
```csharp
using System.Net;
using System.Net.Http;
using Moq;
using Xunit;

namespace UnitTestSamples.Tests
{
    public class ActivityGeneratorTests
    {
        [Fact]
        public void GetActivity_ValidResponse_ReturnsActivity()
        {
            // Arrange

            // This is the json response we want the API to return
            var responseJson = @"{
                ""activity"": ""Make a new friend"",
                ""type"": ""social"",
                ""participants"": 1,
                ""price"": 0,
                ""link"": """",
                ""key"": ""1000000"",
                ""accessibility"": 0
            }";

            // This is the HTTP response message class we want the API to
            // return
            var httpResponseMessage = new HttpResponseMessage(HttpStatusCode.OK);
            httpResponseMessage.Content = new StringContent(responseJson);
            
            // Here, we setup a mock ApiClient instance and program it to 
            // return our canned HTTP response above.
            var apiClient = Mock.Of<IApiClient>();
            Mock.Get(apiClient)
                .Setup(m => m.GetAsync(It.IsAny<string>()))
                .ReturnsAsync(httpResponseMessage);
            
            // We instantiate the SUT and pass in the mock dependency
            var activityGenerator = new ActivityGenerator(apiClient);

            // Act
            var activity = activityGenerator.GetActivity();

            // Assert
            Assert.True(!string.IsNullOrEmpty(activity));
        }
    }
}
```

The `Mock` static class exposes helper methods to create a mock object (`Mock.Of<InterfaceToMock>`), get a reference to the mock setup class (`Mock.Get`), setup a method on the interface to mock (`Mock.Setup`) to return a canned object (`Mock.ReturnsAsync`). The `It` static class lets us specify expectations on input parameters. `It.IsAny<string>()` means any string value is OK.

Now, what if we want to test a scenario where the API call throws an exception? We setup the API to throw a `WebException`, and test the type of the return to be `AggregateException` (since this is [async](https://docs.microsoft.com/en-us/archive/msdn-magazine/2014/october/async-programming-introduction-to-async-await-on-asp-net) code, the underlying exception would be wrapped in an `AggregateException`), and the type of the base exception to be `WebException`:
```csharp
[Fact]
public void GetActivity_APIThrowsException_RaisesException()
{
    // Arrange
    // Here, we setup a mock ApiClient instance and program it to 
    // throw a WebException.
    var apiClient = Mock.Of<IApiClient>();
    Mock.Get(apiClient)
        .Setup(m => m.GetAsync(It.IsAny<string>()))
        .ThrowsAsync(
            new WebException("network error", WebExceptionStatus.ConnectFailure));

    // We instantiate the SUT and pass in the mock dependency
    var activityGenerator = new ActivityGenerator(apiClient);

    // Act
    var ex = Assert.Throws<AggregateException>(()=> activityGenerator.GetActivity());

    // Assert
    Assert.IsType<WebException>(ex.GetBaseException());
}
```

What if we wanted to test that we handle a 500 error from the API gracefully? We could do it like this:
```csharp
[Fact]
public void GetActivity_500Response_ReturnsNull()
{
    // Arrange

    // This is the HTTP response message class we want the API to
    // return
    var response = new HttpResponseMessage(HttpStatusCode.InternalServerError);

    // Here, we setup a mock ApiClient instance and program it to 
    // return our canned HTTP response above.
    var apiClient = Mock.Of<IApiClient>();
    Mock.Get(apiClient)
        .Setup(m => m.GetAsync(It.IsAny<string>()))
        .ReturnsAsync(response);

    // We instantiate the SUT and pass in the mock dependency
    var activityGenerator = new ActivityGenerator(apiClient);

    // Act
    var activity = activityGenerator.GetActivity();

    // Assert
    Assert.True(string.IsNullOrEmpty(activity));
}
```

This is just a small flavor of how mocking frameworks can make our lives easier when it comes to writing unit tests. 

#### Code
[https://github.com/cs31415/unit-test-samples](https://github.com/cs31415/unit-test-samples)

#### Takeaways
- [_Moq_](https://github.com/Moq/moq4/wiki/Quickstart) is a mocking framework for C# and the .NET framework
- Mocking frameworks can help us test various hard to test scenarios
- Mocking helps us write tests that run faster by not making network calls
- Mocking forces us to write well-designed code where dependencies are decoupled from the SUT (System Under Test)
- Injecting dependencies into the SUT makes it easy to mock their behavior

Continue to [part 4](/blog/unit-testing-4).