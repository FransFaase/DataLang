# The Data Language

The Data programming language is focused on data and data manipulation. It is based on ideas I earlier expressed on [The Art of Programming](https://www.iwriteiam.nl/AoP.html). The language will not focus on performance, some kind of execution model, some kind of typing system, but on a consisted representation and manipulation of data. In this respect it is maybe similar to [Rust](https://en.wikipedia.org/wiki/Rust_(programming_language)). I also has some resemblance with [MUMPS Programming Language](https://en.wikipedia.org/wiki/MUMPS). The idea is that ultimately, the language will have concepts like transactions, revision histories, users, and rights. There could be multiple implementation of it, some of which might have interfaces to a [Git repository](https://en.wikipedia.org/wiki/Git) or a relation database instance, that would appears as a single data 'object' in the language. An initial implementation will be based on an interpreter with a rich data structure, but later implementations might consist of compilers to other languages. The language will not have a fixed set of basic value types, but one that will be extendable based on the underlying implementation models.

In the following, I will focus on the semantics of the language and not on its syntax. I will use a mix of concrete and 'abstract' syntax, borrowing syntax from common languages.

## Values

The basic value types are:

*   The undefined value: `undefined`
*   The null (or void) value: `null`
*   Boolean values: `false` and `true`
*   Integer values.
*   Real values.
*   Identifiers values.
*   Unicode character values.

These values can be combined in more composite values using some types, that will be described briefly in the following sections.

#### List value

The list value represents an ordered collection of values. We use square brackets for list value. List constructor examples are:

```
    []
    [1]
    [true, 0, 1]
```

#### Set value

The set value represent an unordered collection of values, where every value can only occur once. We use curly brackets for set values. Set constructor examples are:

```
    {}
    {1}
    {1, 3, true}
```

#### Map value

The map value represent an unorderd collection of key values to result values, where each key value is unique. We use curly brackets and the colon symbol for map values. Map constructor examples:

```
    {}
    {3:5, 6:true}
```

(When we consider `3:null` to be equal with `3`, than sets and map values are basically the same.)

#### Record value

The record value represents a composite value where its members are identified with identifiers (field names). Record values are basically the same as map values where the keys are identifiers. Record constructor examples are:

``` 
    {x:1}
    {a:1, b:2}
```

## Value variables

Values can be assigned to variables with an assignment statement. For the assignment operator an equal symbol is used, like:

``` 
    a = [1, 2, 3];
    b = a;
```

Now the value variables both contain a private copy of the same value. (An implementation could make use of [lazy copying](https://en.wikipedia.org/wiki/Object_copying#Lazy_copy) for efficiency reasons.)

The language has methods for addressing parts of a value. we use the period for this. An integer value, starting from 0, is used to address a part in a list. The value of the key is used for addressing a value in a map or record value. For example:

``` 
    a = {x:1, z:7};
    b = a.z;
    c = [3, 7, 5];
    d = c.1;
    e = {3:4, 5:7};
    f = e.5;
```

will result the value variables b, d, and f, hold the value 7 after execution.

## Value cursors

A cursor is much like a pointer in a value. Cursors are always associated with the value where they are made from. Cursor can become invalid if the part of the value they point to has been removed from the value. Cursors can either be made from (a part of) a value or another cursor. A cursor can be used to retrieve the actual value inside the value it belongs to and it can be used to modify the value it belongs to. A cursor is created with the operator consisting of a minus character followed by the greater-than character. Examples are:

``` 
    a = [3, 4, [3, 7]];
    b -> a.2;
    b -> b.1;
    b = 8;
```

Will cause the value variable `a` to contain the value `[3, 4, [3, 8]]`.

## Compound with components

Although it is possible to create very large and deeply nested values with the above language constructs, it is not possible to have true references. For example the following statements:

``` 
    a = [3, 4];
    b -> a.1;
    b = [5, a];
```

Will result in `a` getting the value `[3, [5, [3, 4]]]` and not some kind of infinitely large value. To introduce the reference concept, we introduce a compound value with components. Reference can only occur within a compound and to a component of the compound. Components can have components again. We use the keyword `component` to identify a component and the keyword `compound` a dollar symbol to define a compound. When a compound occurs as part of a compound, it is not possible to make references from or to the inner compound. With the following statements a compound with two components is created:

``` 
    a = component {x:3, r:4};
    b = component {x:4, r:false};
    c = compound [a, b];
```

## References

There are three types of references with respect what should be done when the target component in a compound is removed. These are:

*   The reference is made null. We call this a nullable reference.
*   The removal is not allowed. We call this a claiming reference.
*   Some part of the value in which the reference occurs, is removed. We call this an existential reference.

For the last case it is also needed to define the extent of the component that needs to removed. A solution for this is to introduce nullable values, meaning that when the target of a existential reference is removed, it means that the smallest enclosing nullable value is removed. As a continuation of the statements of the previous section, we could add the following statements:

``` 
    a -> c.0;
    b -> c.1;
    b.r = nullable refto a;
    a.r = claiming refto b;
    b.x = nullable {v:refto a};
```

Now when the component where the cursor `a` points to is removed, both the `x` and `r` members of the component where the cursor `b` points to, will get the value `null`.
