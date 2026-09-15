# Remove Entries Conditionally

While removing items conditionally from a list is just filtering, removing entries from a context object it a bit more complex.

Filters only work with lists. And we can transform any context object into a list by using: `get entries(someContext)`. This returns a list of context objects specifying the keys and values of each entry.

```
// expression
get entries(someContext)
// result
[
	{
		"key": "someKey",
		"value": "someValue"
	},
	{
		"key": "anotherKey",
		"value": 5
	}
]
```

## Remove Entries by Keys

The context object might use unique ids as keys. Now, you are tasks with removing all entries which keys are contains in some list. So, we get the entries, filter the resulting list, and then we put everything back together.

```
// multi-line context object
{
	"entries": get entries(someContext),
	"filteredEntries": entries[not(list contains(someList, key))],
	"result": context(filteredEntries)
}.result

// one line
context(get entries(someContext)[not(list contains(someList, key))])
```

If someList is static and hard-coded, you can also use a unary test here:
```
context(get entries(someContext)[not(key in ("a", "b", "c"))])
```

## Remove Entries by Values

Similar to filtering the entries by the value of the key, you can also filter by value.

```
// multi-line context object
{
	"entries": get entries(someContext),
	"filteredEntries": entries[value.number < 4],
	"result": context(filteredEntries)
}.result

// one line
context(get entries(someContext)[value.number < 4])
```

Just make sure to not forget to go into value to access its fields!