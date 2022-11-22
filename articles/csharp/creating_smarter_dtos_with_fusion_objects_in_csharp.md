# Creating smarter DTOs with fusion objects in C#

It is often necessary to transmit data from different entities within our code, anonymous (dynamic) objects destroy any possibility of debugging and therefore the best possibility is to create DTOs (Data Transfer Objects).

With the amount of data that must be transported, the amount of DTOs always grows and creates an environment where they become out of sync with the real models, probably due to a technical failure of the developer in the face of so many classes to take care of.

## What are "fusion objects"?

- In my first attempts (outside of any book, documentation or external article), "fusion objects" are entities that inherit from `Tuple<T1, T2, T3...>` and hides the Item1 and Item2 members allowing the developer to expose only the needed members while keeping a direct relationship with the reference entities.
- In my second validation case, the theory of the topic before - if implemented with Tuples - is invalid due to some reasons that will be explored below with independent, composed objects.

## First Attempt

Let's suppose I have two entities in my database: UserAuthentication (which holds the user's authentication information, including sensitive data) and UserItems (which holds any kind of random item referring to that user).

```cs
class UserAuthentication 
{
    public long Id { get; set; }
    public long ItemsId { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
}
```

```cs
class UserItems
{
    public long Id { get; set; }
    public long UserId { get; set; }
    public short ItemAQuantity { get; set; }
    public short ItemBQuantity { get; set; }
}
```

Now, we have to find a way to get specific fields from these models in a presentable way to the final user - **without exposing sensitive or useless data**.

The common way is making a standard DTO model, like the example:

```cs
class UserSomeActionDTO 
{
    public string Username { get; set; }
    public short ItemAQuantity { get; set; }
    public short ItemBQuantity { get; set; }
}
```

Okay, it's great, isn't it? Just no. Now, let's suppose you need to change the `UserAuthentication` template or the `UserItems`. You will also have to change the DTO model directly as well as all your builds, following a difficult maintenance practice and attention issues that can occur frequently.

With "fusion objects" you can control the types that will compose the object explicitly as well as its quantity (if there are two, three or more types that will compose the DTO) and expose only the necessary fields in a safe way.

```cs
class UserSomeActionDTO : Tuple<UserAuthentication, UserItems> 
{
    //Constructor will call default tuple constructor
    public UserSomeActionDTO(
        UserAuthentication item1, 
        UserItems item2
    ) : base(item1, item2) {}

    //We will overwrite "Item1" and "Item2" field to make them private
    private new readonly UserAuthentication Item1;
    private new readonly UserItems Item2;

    //Now we can expose only the needed fields to that object
    public string Username
    {
        get => Item1.Username;
    }
    public short ItemAQuantity
    {
        get => Item2.ItemAQuantity;
    }
    public short ItemBQuantity
    {
        get => Item2.ItemBQuantity;
    }
}
```

Now, can we simple instance this DTO calling `new UserSomeActionDTO(myUserAuthentication, myUserItems)` and use it immediately? No, there are two core problems with the code.

- There is no way to modify the accessibility property from `Tuple` items to secure the object, this go against basic OOP rules and there is no exception even using the `new` keyword to override the field in C#
- There is no way to properly deserialize the object and work with it in "real-life" web applications or similar due to NullException issues with `System.Text.Json`, this means that *another DTO would be needed to make that functional*

## Composing fusion objects

After the first version of this article, via suggestive implementation of a functional DTO model that takes into account instance commented by [@carlhugon](https://dev.to/carlhugom) we can have a smaller and reliable version without using `Tuple<T1, T2, T3>...`:

```cs
class UserSomeActionDTO
{
    //We are declaring the fields private to prevent data exposure
    private readonly UserAuthentication _item1;
    private readonly UserItems _item2;

    //Constructor that calls the union with two required types
    public UserSomeActionDTO(
        UserAuthentication item1, 
        UserItems item2
    )
    {
        _item1 = item1;
        _item2 = item2;
    }

    //Now we can expose only the needed fields to that object
    public string Username => _item1.Username;
    public short ItemAQuantity => _item2.ItemAQuantity;
    public short ItemBQuantity => _item2.ItemBQuantity;
}
```

With the creation of the model, in relation to standard DTOs that are widely used by devs, we have the following advantages:

- The model doesn't need default instance where each field needs to be manually placed by the dev - in my experience the default instantiation model of DTOs has pretty much always been there and caused a lot of problems and delays in development
- Added security by ensuring that unwanted sensitive data will not be exposed if not explicitly selected
- There is also more organization due to the fact that a change in any model would only require changing the DTO model, instead of each point in the code where it was instantiated
