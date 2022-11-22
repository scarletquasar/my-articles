## Stop validating serial numbers manually 
> 03/01/2022

It is common to use numerical validations every time we are dealing with serial numbers and even numbers that we don't want to be negative and in that context most of the programmers make a lot of mistakes implementing unnecessary functionality that can be circumvented with basic knowledge about the primitive type system.

In this article I will bring and explain three ways to perform the validation of this numeric type.

**1. The worst way**

Unfortunately, the way most used by developers of projects I've already got my hands on is just a manual validation usually followed by a throw or "StatusCode that takes an **int** or **long** as an argument.

```cs
[HttpGet]
public IActionResult GetUserInformation([FromRoute] long idUser)
{
    if(idUser < 0)
        return StatusCode(400, "idUser should be non-negative."}

    return Ok(_userService.GetById(idUser));
}
```

**2. The suspicious way**

This approach may be valid but it is not so interesting since it is about overengineering for something that can be solved in a much simpler way.

```cs
[HttpGet]
public IActionResult GetUser([FromRoute] UserRequest request)
{
    return Ok(_userService.GetById(request.Id));
}
```

```cs
using System.ComponentModel.DataAnnotations;

public class UserRequest 
{
    [Range(0, long.MaxValue)]
    public long Id { get; set; }
}
```

**3. The perfect way**

There is a primitive data type that is not widely used that was inherited from older languages like C and C++, namely ulong, uint or ushort accepting only positive values. In addition to simplifying the process, the use of this data type will significantly decrease the weight of operations when it comes to scheduling.

```cs
[HttpGet]
public IActionResult GetUser([FromRoute] ulong idUser)
{
    return Ok(_userService.GetById(request.Id));
}
```
