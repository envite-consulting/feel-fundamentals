# Conditional Append and Merge

This is an important FEEL tool in your tool box: **How to conditionally append an item to a list or merge an entry in a context object**

## Conditionally Append

Let's say we have a boolean `doAppend`. If it is true, we want to add the item `someItem` to the list `someList`. If it is false, we don't want the list to change.

### Root if else then

One option is to use a root `if then else` syntax like this:

```
if doAppend
then append(someList, someItem)
else someList
```

This solution is easy to understand. If your use case is simple, it's a good approach.

If, at some point, you have another condition and another item to add, this solution becomes convoluted. You would need to go through four different conditions, sometimes add both items, etc.

### Append and Filter

Another solution is to always append something. If `doAppend` is true, you append the item, else you append `null` or some other dummy value. Afterwards, you filter out the unwanted dummy values.

```
append(
	someList,
	if doAppend
	then someItem
	else null
)[item != null]
```

This might affect the rest of the list, but that is unlikely.

As `append()` accepts multiple items, this approach can be extended to more complex use cases.

### Concatenate

My favourite solution is similar to the append and filter approach, but using `concatenate()` instead. Instead of `null` or a dummy value, I can use `[]` which does nothing when concatenating.

```
concatenate(
	someList,
	if doAppend
	then [someItem]
	else []
)
```

If I am only adding a single item, I need to put it into a list like `[someItem]`. But sometimes, I want to conditionally add an entire list to `someList`. And in that case, only the `concatenate()` approach works.

### Summary

| Approach          | Complexity | Flexibility | Stability |
| ----------------- | ---------- | ----------- | --------- |
| Root if then else | low        | low         | high      |
| Append and filter | medium     | medium      | medium    |
| Concatenate       | medium     | high        | high      |

Thus, I am always using the concatenate approach.




## Conditionally Merge

Adding an entry to a context objects works pretty much the same way. There is `context put()` function to put a single entry in a context object, similar to `append()` for lists. But it is not really applicable or useful in this case.

Instead, we directly use the more flexible `context merge()` approach. This function always takes a list of contexts, so don't forget the `[]`!

### Root if then else

Solid root `if then else`:
```
if mergeEntry
then context merge([someContext, someEntry])
else someContext
```

### Context Merge

Flexible approach, with an empty context object `{}` as the dummy value.

```
context merge([
	someContext,
	if mergeEntry
	then someEntry
	else {}
])
```

Again, this approach can be extended to adding multiple entry under multiple conditions.

### Summary

| Approach          | Complexity | Flexibility | Stability |
| ----------------- | ---------- | ----------- | --------- |
| Root if then else | low        | low         | high      |
| Context merge     | medium     | high        | high      |

Thus, I am always using the context merge approach.