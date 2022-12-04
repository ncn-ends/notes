source: https://www.youtube.com/watch?v=cqCBhkNroDI

[String Literals](#new-string-literals)  
[List Patterns](#list-patterns)  
[Generic Attributes](#generic-attributes)  
[UTF-8 String Literals](#utf-8-string-literals)  
[Generic Maths](#generic-maths)  
[Required Members](#required-members)  
[File scoped types](#file-scoped-types)  
[Improved method to delegate conversion](#improved-method-to-delegate-conversion)  

### **New String Literals**

```cs
var json = """;
{
    data: "hello"
}
""";
```
- requires 3 double quotes minimum
- match number of double quotes before with after
- maintains whitespace

```cs
var msg = "world";
var json = $$""""""
{
    "hello": "{msg}"
}
"""""";
```
- can increase number of quotes unlimited times to avoid invalid syntax
- prepend `$$` to avoid invalid syntax with json curly braces

### **List Patterns**

- applies to anything with a count property
    - e.g. list, array, ReadOnlySpan, etc.

```cs
int[] numbers = { 1, 2, 3 };

Console.WriteLine(numbers is [1, 2, 3]); // --> True
Console.WriteLine(numbers is [1, 2, 4]); // --> False
Console.WriteLine(numbers is [1, 2, 3, 4]); // --> False
Console.WriteLine(numbers is [0 or 1, <= 2, >= 3]); // --> True

if (numbers is [var first, _, _]) 
{
    Console.WriteLine(first); // --> 1
}

if (numbers is [var first, .. var rest])
{
    Console.WriteLine(first); // --> 1
    // reset becomes { 2, 3 }
}


static string GetOutput(string[] name) =>
    name switch
    {
        [var fullName] => $"""Name is { fullName} """ ,
        [var firstName, var lastName] => $"""Name is { firstName}  { lastName} """ ,
        _ => "No name found."
    };

Console.WriteLine(GetOutput(new string[] { "Stanley Kubrick" })); // --> Stanley Kubrick
Console.WriteLine(GetOutput(new string[] {"Damien", "Chazelle"})); // -> Damien Chazelle
Console.WriteLine(GetOutput(new string[] {})); // -> No name found

```

### **Generic Attributes**

- previously had to pass type down using `typeof` as an argument in the contructor
- this would require reflection and other complications
```cs
public class ValidatorAttribute : Attribute
{
    public Type ValidatorType { get; }

    public ValidatorAttribute(Type type)
    {
        ValidatorType = type;
    }
}
```
```cs
[Validator(typeof(UserValidator))]
public class User
{

}
```

- allows you to avoid being able to pass in types that don't implement IValidator to the Validator argument
- no more need for reflection (?)
```cs
public class ValidatorAttribute<TValidator> : Attribute : TValidator : IValidator
{
    public Type ValidatorType { get; }

    public ValidatorAttribute(Type type)
    {
        ValidatorType = typeof(TValidator);
    }
}
```
```cs
[Validator<UserValidator>]
public class User
{

}
```

### **UTF-8 String Literals**
- C# strings by default are utf-18, whereas the web runs on utf-8
    - can optimize by avoiding having to convert back and forth

```cs
ReadOnlySpan<byte> text = Encoding.Unicode.GetBytes("Quentin Tarantino"); // 34 bytes
ReadOnlySpan<byte> text = "Quentin Tarantino"u8; // 17 bytes
```

### **Generic Maths**

- This method works for int, but what if you wanted one for doubles?
- Previously you would have to create another method with a differnet signature to account for doubles, and then many other number types
```cs
int AddAll(int[] nums)
{
    int res = 0;
    foreach (var num in nums)
    {
        res += num;
    }

    return res;
}
```

- Now you can use generics
- T will have many static properties to take advantage of, this is just one example
```cs
T AddAll<T>(T[] nums) where T : INumber<T>
{
    T res = T.Zero;
    foreach (var num in nums)
    {
        res += num;
    }

    return res;
}
```

### **Required Members**
- Previously the `init` keyword on a property prevented you from setting the property after it's been instantiated, but it didn't prevent you from forgetting to set the property when instinatiating the object.
```cs
var user = new User(); // no error

class User 
{
    public string FullName {get; init; }
}
```

- The required keyword forces you to provide a value when instantiating it
```cs
var userA = new User(); // error
var userB = new User
{
    FullName = "Peter Jackson";
}; // no error
userB.FullName = "Peter Parker"; // error

class User 
{
    public required string FullName {get; init; }
}
```

### **File scoped types**
- In addition to other access modifiers for types, like `internal`, `public`, etc. you can now also use `file`
    - This restricts the usage of the type only to that specific file

### **Improved method to delegate conversion**

- Previously if you passed the method argument directly like the Sum method, it would be worse performance and memory usage than passing it like the Multiple method does, even though the end result is the same
- With C#11 it's the same overall
```cs
public int Sum(int[] nums)
{
    return nums.Where(Filter).Sum(); // slower / worse memory usage before C#11
}

public int Sum(int[] nums)
{
    return nums.Where(x => Filter(x)).Sum(); // better before C#11
}

static bool Filter(int age)
{
    return age > 50;
}
```