# Running tests in parallel with xunit

source: https://matthewregis.dev/posts/running-tests-in-parallel-with-xunit

*Published on Saturday, 30 December 2023*

 [csharp](/tags/csharp) [dotnet](/tags/dotnet) [dotnet 7](/tags/dotnet-7) [unit tests](/tags/unit-tests) [xunit](/tags/xunit) 

Introduction
============

Running tests in parallel offers several advantages that contribute to the efficiency and effectiveness of the software development process, without going too much into depth of the benefits of running tests in parallel, here's are short list of why you might want to execute tests in parallel:

- Reduced Execution Time
- Faster Feedback Loop
- Optimized Resource Utilization
- Increased Test Coverage
- Efficient Continuous Integration (CI) Pipelines
- Scalability
- Improved Developer Productivity

In this blog post I'll be exploring executing tests in parallel with xUnit.

xUnit
=====

xUnit offers running test in parallel out of the box if your using the latest version (version 2). xUnit identifies test collections, and tests within a collection are executed sequentially, but different collections can run in parallel.

Tests separated in different test classes are treated as separate collections, but you still have the flexibility identify tests in different classes as part of the same collection.

For more information, here's the official documentation [running-tests-in-parallel](https://xunit.net/docs/running-tests-in-parallel).

Sequential tests
----------------

Tests in the following example will run sequentially.

```csharp
public class TestClass1
{
    [Fact]
    public void Test1()
    {
        Thread.Sleep(3000);
    }

    [Fact]
    public void Test2()
    {
        Thread.Sleep(5000);
    }
}

```

> ![](/posts/images/xunit-sequential-tests.gif)

Parallel tests
--------------

Tests in different classes are treated as separate collections and are run in parallel.

```csharp
public class TestClass1
{
    [Fact]
    public void Test1()
    {
        Thread.Sleep(3000);
    }
}

public class TestClass2
{
    [Fact]
    public void Test2()
    {
        Thread.Sleep(5000);
    }
}

```

> ![](/posts/images/xunit-parallel-tests.gif)

Sequential tests using the `[Collection]` attribute.
----------------------------------------------------

Although the following tests are separated into different classes which would normally run in parallel, you can change this behavior by adding the `[Collection]` attribute with the same name. Now tests will execute sequentially.

```csharp
[Collection("Our Test Collection #1")]
public class TestClass1
{
    [Fact]
    public void Test1()
    {
        Thread.Sleep(3000);
    }
}

[Collection("Our Test Collection #1")]
public class TestClass2
{
    [Fact]
    public void Test2()
    {
        Thread.Sleep(5000);
    }
}

```

> ![](/posts/images/xunit-collection-tests.gif)

Parallel mixed with sequential tests plus
-----------------------------------------

There maybe some tests that you want to run sequentially and away from the collection of parallel tests. This could be because the test in question alters or affects the result of other tests. This is also achievable by using the `[CollectionDefinition("CollectionName", DisableParallelization = true)]` attribute like below.

```csharp
public class TestClass1
    {
        [Fact]
        public void Test1()
        {
            Thread.Sleep(3000);
        }
    }

    public class TestClass2
    {
        [Fact]
        public void Test2()
        {
            Thread.Sleep(5000);
        }
    }

    [CollectionDefinition(nameof(TestClass3), DisableParallelization = true)]
    [Collection(nameof(TestClass3))]
    public class TestClass3
    {
        [Fact]
        public void Test3()
        {
            Thread.Sleep(3000);
        }
    }

```

> ![](/posts/images/xunit-parrallel-with-sequential-tests.gif)

ParallelTestFramework
=====================

The [Meziantou.Xunit.ParallelTestFramework](https://github.com/meziantou/Meziantou.Xunit.ParallelTestFramework) nuget package builds upon xUnit and adds parallelization for each test within a collection. It is worth noting to proceed with caution if your tests have shared state, as tests might not behave in the deterministic way you expect.

The beauty of this package is it uses the parallelization built within dotnet, more specifically the [`Task.WhenAll` method](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.whenall?view=net-8.0). Which you can see for yourself in the [`ParallelTestClassRunner.cs`](https://github.com/meziantou/Meziantou.Xunit.ParallelTestFramework/blob/main/Meziantou.Xunit.ParallelTestFramework/ParallelTestClassRunner.cs) class.

There isn't much configuration needed to enable this package, simply install the package and the tests should then run parallel. To add the package run the following command.

```ps
dotnet add package Meziantou.Xunit.ParallelTestFramework

```

Code Example
============

> ***NOTE:*** All the code for this section can be seen at this [github repository](https://github.com/reggieray/dotnet-parallel-tests).

In this section I'll put everything together from what I mentioned above. First I created a test project. I followed the [Unit testing C# in .NET Core using dotnet test and xUnit](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test) article from Microsoft learn website to get going.

The project comprises of dotnet library and a test project. I updated this slightly, I added more tests and updated the `IsPrime` method to have a delay to highlight the differences in timing when running tests.

The code
--------

dotnet library code in `PrimeService.cs`.

```csharp
﻿namespace Prime.Services
{
    public class PrimeService
    {
        public async Task IsPrime(int candidate)
        {
            await Task.Delay(5000);

            if (candidate == 1)
            {
                return false;
            }

            if (candidate == 2)
            {
                return true;
            }

            if (candidate == 3)
            {
                return true;
            }

            throw new NotImplementedException("Not fully implemented.");
        }
    }
}

```

dotnet test code:

`PrimeService_IsPrimeShould.cs`

```csharp
namespace PrimeService.Tests;

public class PrimeService_IsPrimeShould
{
    [Fact]
    public async Task IsPrime_InputIs1_ReturnFalse()
    {
        var primeService = new Prime.Services.PrimeService();
        bool result = await primeService.IsPrime(1);

        Assert.False(result, "1 should not be prime");
    }

    [Fact]
    public async Task IsPrime_InputIs2_ReturnTrue()
    {
        var primeService = new Prime.Services.PrimeService();
        bool result = await primeService.IsPrime(2);

        Assert.True(result, "2 should be prime");
    }

    [Fact]
    public async Task IsPrime_InputIs3_ReturnTrue()
    {
        var primeService = new Prime.Services.PrimeService();
        bool result = await primeService.IsPrime(3);

        Assert.True(result, "3 should be prime");
    }
}

```

`PrimeService_IsPrimeShouldThrow.cs`

```csharp
namespace PrimeService.Tests;

public class PrimeService_IsPrimeShouldThrow
{
    [Fact]
    public async Task IsPrime_InputIs4_ReturnNotImplementedException()
    {
        var primeService = new Prime.Services.PrimeService();
        await Assert.ThrowsAsync(() => primeService.IsPrime(4));
    }

    [Fact]
    public async Task IsPrime_InputIs5_ReturnNotImplementedException()
    {
        var primeService = new Prime.Services.PrimeService();
        await Assert.ThrowsAsync(() => primeService.IsPrime(5));
    }

    [Fact]
    public async Task IsPrime_InputIs6_ReturnNotImplementedException()
    {
        var primeService = new Prime.Services.PrimeService();
        await Assert.ThrowsAsync(() => primeService.IsPrime(6));
    }
}

```

Test explorer
-------------

To show the visual difference in execution time here are two gif's. The gif below is just using xUnit.

> ![](/posts/images/xunit.gif)

The following gif shows the difference after adding the [Meziantou.Xunit.ParallelTestFramework](https://github.com/meziantou/Meziantou.Xunit.ParallelTestFramework) nuget package.

> ![](/posts/images/xunit-2.gif)

Continuous integration
----------------------

It is also important to know if tests run in parallel in a continuous integration (CI) environment. I have added a github action to the github repo to show case this which you can find [here](https://github.com/reggieray/dotnet-parallel-tests/actions).

The following is the output run before adding the package, you can see the original output [here](https://github.com/reggieray/dotnet-parallel-tests/actions/runs/7365270219/job/20046431006). The total execution time was 17.7734 Seconds.

```ps
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
[xUnit.net 00:00:00.00] xUnit.net VSTest Adapter v2.4.5+1caef2f33e (64-bit .NET 7.0.14)
[xUnit.net 00:00:04.02]   Discovering: PrimeService.Tests
[xUnit.net 00:00:04.14]   Discovered:  PrimeService.Tests
[xUnit.net 00:00:04.15]   Starting:    PrimeService.Tests
[xUnit.net 00:00:09.24]   Finished:    PrimeService.Tests
  Passed PrimeService.Tests.PrimeService_IsPrimeShouldThrow.IsPrime_InputIs6_ReturnNotImplementedException [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShould.IsPrime_InputIs1_ReturnFalse [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShouldThrow.IsPrime_InputIs4_ReturnNotImplementedException [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShould.IsPrime_InputIs3_ReturnTrue [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShould.IsPrime_InputIs2_ReturnTrue [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShouldThrow.IsPrime_InputIs5_ReturnNotImplementedException [5 s]

Test Run Successful.
Total tests: 6
     Passed: 6
 Total time: 17.7734 Seconds
     1>Done Building Project "/home/runner/work/dotnet-parallel-tests/dotnet-parallel-tests/dotnet-parallel-tests.sln" (VSTest target(s)).

```

The output next was taken from this [github action run](https://github.com/reggieray/dotnet-parallel-tests/actions/runs/7366546892/job/20048916904). Which shows a execution time of 6.4073 Seconds, which is a difference of around 90%.

```ps
Starting test execution, please wait...
A total of 1 test files matched the specified pattern.
[xUnit.net 00:00:00.00] xUnit.net VSTest Adapter v2.4.5+1caef2f33e (64-bit .NET 7.0.14)
[xUnit.net 00:00:00.52]   Discovering: PrimeService.Tests
[xUnit.net 00:00:00.56]   Discovered:  PrimeService.Tests
[xUnit.net 00:00:00.56]   Starting:    PrimeService.Tests
[xUnit.net 00:00:05.64]   Finished:    PrimeService.Tests
  Passed PrimeService.Tests.PrimeService_IsPrimeShouldThrow.IsPrime_InputIs6_ReturnNotImplementedException [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShould.IsPrime_InputIs1_ReturnFalse [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShould.IsPrime_InputIs2_ReturnTrue [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShould.IsPrime_InputIs3_ReturnTrue [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShouldThrow.IsPrime_InputIs4_ReturnNotImplementedException [5 s]
  Passed PrimeService.Tests.PrimeService_IsPrimeShouldThrow.IsPrime_InputIs5_ReturnNotImplementedException [5 s]

Test Run Successful.
Total tests: 6
     Passed: 6
 Total time: 6.4073 Seconds
     1>Done Building Project "/home/runner/work/dotnet-parallel-tests/dotnet-parallel-tests/dotnet-parallel-tests.sln" (VSTest target(s)).

```

Links
=====

- [xUnit.net - Running Tests in Parallel](https://xunit.net/docs/running-tests-in-parallel)
- [Meziantou.Xunit.ParallelTestFramework](https://github.com/meziantou/Meziantou.Xunit.ParallelTestFramework)
- [Example github repository](https://github.com/reggieray/dotnet-parallel-tests)
- [Microsoft Learn - Unit testing C# in .NET Core using dotnet test and xUnit](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test)
