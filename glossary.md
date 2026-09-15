# Glossary

## Context Object
In FEEL, a collecting of key-value pairs with unique keys is called a `context object`. Next to basic data types, temporal data types and lists, `context objects` are arguably the most useful data type of all.

While the keys of a `context object` are always strings, the values can be anything, including a FEEL expression resulting in some other data type, e.g., a boolean, number, list, another `context object` or even a custom function.

While it is possible to omit the double quotes around the keys, I strongly suggest to always add them explicitely.

```
{
    "writeFullName": function(personalData) personalData.firstName + " " + personalData.lastName,
    "result": string join(
        for personalData 
        in guestList 
        return writeFullName(personalData),
        ",\n"
    )
}.result
```

A `context object` can be used to explicitely pipe multiple calculation steps instead of using a nested function calls. In the example above, a function is defined to separate the logic how a full name is concatenated from personalData. The `result` is the desired output of this FEEL expression. It is calculated and then returned via `.result`. This schema is very useful to make FEEL expressions more readable.

## Execution Context
Most FEEL expressions use or manipulate existing values. These values are provided to the FEEL expression via the `execution context`. Some examples include:

* process variables and local varibales are the `execution context` when working with script tasks or input/output mappings in BPMN
* process variables and local variables are the `execution context` when evaulating a `unary test` in a DMN table cell

The `execution context` is a JSON object. Variables can be directly accessed via `jungle` and directly manipulated, e.g. via a `projection` — `jungle.type` — which creates a list of types.
```jsonc
{
    "jungle": [
        {
            "type": "snake",
            "mood": "depressed"
            //...
        },
        {
            "type": "parrot",
            "mood": "grumpy"
            //...
        }
        //...
    ]
    //...
}
```

## Iterator Variable
As in many other languages, an `iterator variable` is defined in FEEL when looping over multiple items in a list. This `iterator variable` is only available in the scope of the for loop.

```                 
some animal
in jungle
satisfies 
  animal.type = "parrot" 
  and 
  animal.mood = "grumpy"
```

In this example, `animal` is a the `iterator variable` and local to the for loop.

## Unary Test
There are two modes in FEEL: expression and unary test. 

The expression uses the `execution context` as input variables and has any data type as a result

A unary test uses an implicit input variable, as well as the `execution context`. The result is a boolean. The implicit input variable is tested against a provided unary test.

Example: 
* implicit input: x (has value 5 during runtime)
* unary test: > 3
* result during runtime: true

You can use functions in unary tests. In that case, the implicit input can be specified with `?`: `list contains(?, 3)`. In this case, the implicit input is a list. And it is tested to contain the value 3.

On the other hand, you can also use unary tests in expressions by using the keyword `in`: someVariable in > 3. Here someVariable becomes the (not so) implicit input for the unary test.

I like to use unary tests for null checks. See FEEL snippet no. 2!