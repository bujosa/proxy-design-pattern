# Proxy Design Pattern in Go
The proxy design pattern provides a surrogate or placeholder for another object to control access to it. This pattern involves a single class, called proxy, which represents the functionality of another class.

In this Go code, we have implemented a proxy for user management. The `UserListProxy` struct is the proxy that controls access to the `UserList` struct.

# Code Explanation

## User Struct 

```go
type User struct {
    ID int32
}
```

The `User` struct represents a user with an ID of type `int32`.

## UserFinder Interface

```go
type UserFinder interface {
    FindUser(id int32) (User, error)
}
```

The `UserFinder` interface defines a method `FindUser` that all structs implementing this interface should have.

## UserList Struct

```go
type UserList []User
```

The `UserList` struct is a slice of `User` structs. It implements the `UserFinder` interface.

## FindUser Method

```go
func (t *UserList) FindUser(id int32) (User, error) {
    for i := 0; i < len(*t); i++ {
        if (*t)[i].ID == id {
            return (*t)[i], nil
        }
    }

    return User{}, fmt.Errorf("Could not find user with id: %d", id)
}
```

The `FindUser` method iterates over the `UserList` to find a user with the given ID. If it can't find the user, it returns an error.

## UserListProxy Struct

```go
type UserListProxy struct {
    MockedDatabase      *UserList
    StackCache          UserList
    StackSize           int
    LastSearchUsedCache bool
}
```
The `UserListProxy` struct is the proxy for `UserList`. It has a `MockedDatabase` which is the actual `UserList`, a `StackCache` which is a cache of users, a `StackSize` which is the maximum size of the cache, and a `LastSearchUsedCache` which is a flag indicating whether the last search used the cache.

## FindUser Method

```go
func (u *UserListProxy) FindUser(id int32) (User, error) {
    //Search for the object in the cache list first
    user, err := u.StackCache.FindUser(id)
    if err == nil {
        fmt.Println("Returning user from cache")
        u.LastSearchUsedCache = true
        return user, nil
    }

    //Object is not in the cache list. Search in the heavy list
    user, err = u.MockedDatabase.FindUser(id)
    if err != nil {
        return User{}, err
    }

    //Adds the new user to the stack, removing the last if necessary
    u.addUserToStack(user)

    fmt.Println("Returning user from database")
    u.LastSearchUsedCache = false
    return user, nil
}
```

The `FindUser` method in `UserListProxy` first searches for the user in the cache. If it can't find the user in the cache, it searches in the `MockedDatabase`. If it finds the user in the `MockedDatabase`, it adds the user to the cache.