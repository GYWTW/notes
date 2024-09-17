[April 5, 2020](https://hamidmosalla.com/2020/04/05/xunit-part-8-using-theorydata-instead-of-memberdata-and-classdata/) by [Hamid Mosalla](https://hamidmosalla.com/author/hamidmosalla/) | [.Net](https://hamidmosalla.com/category/dotnet/), [XUnit](https://hamidmosalla.com/category/xunit/) in [ClassData](https://hamidmosalla.com/tag/classdata/), [MemberData](https://hamidmosalla.com/tag/memberdata/), [TheoryData](https://hamidmosalla.com/tag/theorydata/), [xUnit](https://hamidmosalla.com/tag/xunit/)

[XUnit – Part 8: Using TheoryData Instead of MemberData and ClassData](https://hamidmosalla.com/2020/04/05/xunit-part-8-using-theorydata-instead-of-memberdata-and-classdata/)
==============================================================================================================================================================================

ref: https://hamidmosalla.com/2020/04/05/xunit-part-8-using-theorydata-instead-of-memberdata-and-classdata/

I previously wrote about using [MemberData, ClassData in this post](https://hamidmosalla.com/2017/02/25/xunit-theory-working-with-inlinedata-memberdata-classdata/). The problem with those are their reliance on `IEnumerable<object[]>`. This can cause runtime issues even if we don’t get any errors at compile time. In this post I’m going to introduce a strongly typed option in xUnit called TheoryData. So in this post I’m going to re-implement the things I did in my previous post with MemberData and ClassData. Except this time I’m going to use TheoryData.

Using TheoryData Instead of ClassData and MemberData
----------------------------------------------------

In this section I’m going to replace the code that I’ve written before with ClassData and Member Data to use the theory data instead.

### Replacing ClassData with TheoryData

Previously I used ClassData like this, I’m going to convert this code to use TheoryData instread.

```cs
public class TestDataGenerator : IEnumerable<object[]>
{
    private readonly List<object[]> _data = new List<object[]>
    {
        new object[] {5, 1, 3, 9},
        new object[] {7, 1, 5, 3}
    };

    public IEnumerator<object[]> GetEnumerator() => _data.GetEnumerator();

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

public class ParameterizedTests
{
    public bool IsOddNumber(int number)
    {
        return number % 2 != 0;
    }

    [Theory]
    [ClassData(typeof(TestDataGenerator))]
    public void AllNumbers_AreOdd_WithClassData(int a, int b, int c, int d)
    {
        Assert.True(IsOddNumber(a));
        Assert.True(IsOddNumber(b));
        Assert.True(IsOddNumber(c));
        Assert.True(IsOddNumber(d));
    }
}
```

We can rewrite the same thing using the TheoryData like so.

```cs
public class TestDataGeneratorUsingTheoryData : TheoryData<int, int, int, int>
{
    public TestDataGeneratorUsingTheoryData()
    {
        Add(5, 1, 3, 9);
        Add(7, 1, 5, 3);
    }
}

public class ParameterizedTestsWithTheoryData
{
    public bool IsOddNumber(int number)
    {
        return number % 2 != 0;
    }

    [Theory]
    [ClassData(typeof(TestDataGeneratorUsingTheoryData))]
    public void AllNumbers_AreOdd_WithTheoryData(int a, int b, int c, int d)
    {
        Assert.True(IsOddNumber(a));
        Assert.True(IsOddNumber(b));
        Assert.True(IsOddNumber(c));
        Assert.True(IsOddNumber(d));
    }
}
```

Doing things like this has many advantages. First of all, it’s simple and less code to write. Second it’s strongly-typed code which is going to give us compile time error if anything in wrong in our code.

[![](https://hamidmosalla.com/wp-content/uploads/2020/04/05.04.2020_20.41.48_REC.png)](https://hamidmosalla.com/wp-content/uploads/2020/04/05.04.2020_20.41.48_REC.png)

We can see that if we try to add the string instead of the expected type we receive a compile time error. This was not he case when we’ve used ClassData because then we were relying on object to provide data.

### Replacing MemberData with TheoryData

Previously I also used MemberData for provide data to my unit tests. MemberData was simpler in that it didn’t need for a new class to be created.  

```cs
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public class TestDataGenerator : IEnumerable<object[]>
{
    public static IEnumerable<object[]> GetNumbersFromDataGenerator()
    {
        yield return new object[] { 5, 1, 3, 9 };
        yield return new object[] { 7, 1, 5, 3 };
    }

    public static IEnumerable<object[]> GetPersonFromDataGenerator()
    {
        yield return new object[]
        {
            new Person {Name = "Tribbiani", Age = 56},
            new Person {Name = "Gotti", Age = 16},
            new Person {Name = "Sopranos", Age = 15},
            new Person {Name = "Corleone", Age = 27}
        };

        yield return new object[]
        {
            new Person {Name = "Mancini", Age = 79},
            new Person {Name = "Vivaldi", Age = 16},
            new Person {Name = "Serpico", Age = 19},
            new Person {Name = "Salieri", Age = 20}
        };
    }
}

public class ParameterizedTests
{
    public bool IsOddNumber(int number)
    {
        return number % 2 != 0;
    }

    public bool IsAboveFourteen(Person person)
    {
        return person.Age > 14;
    }

    public static IEnumerable<object[]> GetNumbers()
    {
        yield return new object[] { 5, 1, 3, 9 };
        yield return new object[] { 7, 1, 5, 3 };
    }

    [Theory]
    [MemberData(nameof(GetNumbers))]
    public void AllNumbers_AreOdd_WithMemberData(int a, int b, int c, int d)
    {
        Assert.True(IsOddNumber(a));
        Assert.True(IsOddNumber(b));
        Assert.True(IsOddNumber(c));
        Assert.True(IsOddNumber(d));
    }

    [Theory]
    [MemberData(nameof(TestDataGenerator.GetNumbersFromDataGenerator), MemberType = typeof(TestDataGenerator))]
    public void AllNumbers_AreOdd_WithMemberData_FromDataGenerator(int a, int b, int c, int d)
    {
        Assert.True(IsOddNumber(a));
        Assert.True(IsOddNumber(b));
        Assert.True(IsOddNumber(c));
        Assert.True(IsOddNumber(d));
    }

    [Theory]
    [MemberData(nameof(TestDataGenerator.GetPersonFromDataGenerator), MemberType = typeof(TestDataGenerator))]
    public void AllPersons_AreAbove14_WithMemberData_FromDataGenerator(Person a, Person b, Person c, Person d)
    {
        Assert.True(IsAboveFourteen(a));
        Assert.True(IsAboveFourteen(b));
        Assert.True(IsAboveFourteen(c));
        Assert.True(IsAboveFourteen(d));
    }
}
```

But with MemberData we still relied on object, not to mention that the code is not as intuitive. Let’s convert the code above to the code that uses TheoryData instead.

```cs
public class ReplacingMemberDataUsingTheoryData
{
    public bool IsAboveFourteen(Person person)
    {
        return person.Age > 14;
    }

    public static TheoryData<Person> PersonData =>
        new TheoryData<Person>
        {
            new Person {Name = "Tribbiani", Age = 56},
            new Person {Name = "Gotti", Age = 16},
            new Person {Name = "Sopranos", Age = 15},
            new Person {Name = "Corleone", Age = 27},
            new Person {Name = "Mancini", Age = 79},
            new Person {Name = "Vivaldi", Age = 16},
            new Person {Name = "Serpico", Age = 19},
            new Person {Name = "Salieri", Age = 20}
        };

    [Theory]
    [MemberData(nameof(PersonData), MemberType = typeof(ReplacingMemberDataUsingTheoryData))]
    public void AllPersons_AreAbove14_WithTheoryData_FromDataGenerator(Person a)
    {
        Assert.True(IsAboveFourteen(a));
    }
}
```

As you can see in the code above, this approach is much more intuitive and short. Also we get the advantages of compile time checking when we want to fill in our data for our tests which means even more accurate tests. I personally will not going to use ClassData and MemberData. Instead I’m going to use TheoryData that as I’ve mentioned has a lot of advantages over those two. Andrew Lock also has a post about this which can be found [here](https://andrewlock.net/creating-strongly-typed-xunit-theory-test-data-with-theorydata/).
