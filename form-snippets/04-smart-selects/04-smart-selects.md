# Smart Select Components

## The Issue

You are tasked with finding a four-letter name for your pet, but you're super bad at picking names.

No worries! Let's build a Camunda Form for this!

## The Solution

From a given pool of four-letter names (see input_data.json), the form will present 4 select components for you.
With the first select component, you will select the first letter of the name. With the second the second, and so forth.

### The Simple Approach

It is not that hard to configure the option expressions for the four select components. Just filter the names by matching substrings, like:

```
// option expression for the third select component
names[substring(item, 1, 1) = first and substring(item, 2, 1) = second]
```

The issues with this is: performance!

You won't notice it with a small set of options and the trivial data we are working with. But imagine a large set of complex data. Filtering by various fields can take time. Even in the fourth component, you filter the entire raw data!

### Introducing a FEEL Component

The solution is quite straight-forward: a FEEL component to extract the computation and to dumb-down the select components!

It might look like this:

```
{
  "secondEntries": names[substring(item, 1, 1) = first],
  "secondOptions": 
	distinct values(
      for name
      in secondEntries
      return substring(name, 2, 1)
    ),
  "thirdEntries": secondEntries[substring(item, 2, 1) = second],
  "thirdOptions": 
	distinct values(
      for name
      in thirdEntries
      return substring(name, 3, 1)
    ),
  "fourthEntries": thirdEntries[substring(item, 3, 1) = third],
  "fourthOptions": 
	distinct values(
      for name
      in fourthEntries
      return substring(name, 4, 1)
    )
}
```

On form change, this data recomputes. `secondEntries` will take the longest. But `secondEntries` itself is already much smaller than the raw data, probably about 95% smaller in our case. Computing `thirdEntries` and `fourthEntires` will be super fast!

And what about `firstOptions`? Well, you computed that one in your workflow, because it has no dependencies, see input_data.json!

### Introducing a Function to Create Options

You might have noticed that the logic to create the options is repetitive. From a given list, get the distinct values of the list of letters that are found at a specific position in the name. We can also add a sort function to the mix.

Similarly, creating the entries is also about comparing the letter of a name at a position with a given letter.

Thus, we can simplify our expression:

```
{
  // functions
  "createEntries": function(nameList, position, letter)
    nameList[substring(item, position, 1) = letter],
  "createOptions": function(nameList, position) 
    sort(
      distinct values(
        for name 
        in nameList 
        return substring(name, position, 1)
      ), 
      function(a, b) a < b
    ),

  // data
  "secondEntries": createEntries(names, 1, first),
  "secondOptions": createOptions(secondEntries, 2),
  "thirdEntries": createEntries(secondEntries, 2, second),
  "thirdOptions": createOptions(thirdEntries, 3),
  "fourthEntries": createEntries(thirdEntries, 3, third),
  "fourthOptions": createOptions(fourthEntries, 4),
}
```


### Even Smarter

It can happen that the third letter is already fixed by selecting the first and second letter. If you pick "D" and "r", there is only "Drew" left.

With our current solution, the user still needs to pick the third and fourth letter to get the result. But we can use the FEEL component for this as well. Let's introduce `oneSecond`, `oneThird` and `oneFourth`. If there is only one item in the options list, we settle on that one directly. I introduced another function for that.

```
{
  // functions
  "createEntries": function(nameList, position, letter)
    nameList[substring(item, position, 1) = letter],
  "createOptions": function(nameList, position) 
    sort(
      distinct values(
        for name 
        in nameList 
        return substring(name, position, 1)
      ), 
      function(a, b) a < b
    ),
  "firstEntryOrNull": function(someList) if count(someList) = 1 then someList[1] else null,

  // data
  "secondEntries": createEntries(names, 1, first),
  "secondOptions": createOptions(secondEntries, 2),
  "oneSecond": firstEntryOrNull(secondOptions),
  "thirdEntries": createEntries(secondEntries, 2, if oneSecond != null then oneSecond else second),
  "thirdOptions": createOptions(thirdEntries, 3),
  "oneThird": firstEntryOrNull(thirdOptions),
  "fourthEntries": createEntries(thirdEntries, 3, if oneThird != null then oneThird else third),
  "fourthOptions": createOptions(fourthEntries, 4),
  "oneFourth": firstEntryOrNull(fourthOptions)
}
```

Picking "D" and "r" now results in `oneThird = "e"` and `oneFourth = "w"`. We already have our result after just selecting our first two letters!

### More Features

I added the following features to my form (04-smart-selects.form):

* make the component read-only if only one option remains
  * potential for improvement: hide the component and replace it with an html component looking like a read-only component with the remaining option displayed
* displaying "not unique yet" or the result to the user
* displaying the remaining options 