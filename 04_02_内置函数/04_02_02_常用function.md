
# 1 lookup

https://developer.hashicorp.com/terraform/language/functions/lookup

==lookup retrieves the value of a single element from a map==, given its key. If the given key does not exist, the given default value is returned instead.


lookup(map, key, default)

For historical reasons, the `default` parameter is actually optional. However, omitting `default` is deprecated since v0.7 because that would then be equivalent to the native index syntax, `map[key]`.

```
> lookup({a="ay", b="bee"}, "a", "what?")
ay
> lookup({a="ay", b="bee"}, "c", "what?")
what?

> lookup({a="hello", b="goodbye"}, "c", "what?")
>what?
```




# 2 file 
file reads the contents of a file at the given path and returns them as a string.

> file("${path.module}/hello.txt")
Hello World



# 3 templatefile

https://developer.hashicorp.com/terraform/language/functions/templatefile
`templatefile` reads the file at the given path and renders its content as a template using a supplied set of template variables.

templatefile(path, vars)


![](image/Pasted%20image%2020231117225551.png)


# 4 slice 

slice(list, from, to) - Returns the portion of list between from (inclusive) and to (exclusive).

For example - if we have: `[a, bb, ccc, dddd, eeeee]`
How can the first 3 elements be selected: a, bb, ccc
And then the 4th and 5th elements: dddd, eeeee
slice(<put_reference_to_list_here, 0, 3)
slice(<put_reference_to_list_here, 3, 5)


# 5 index 

> `index(["a", "b", "c"], "c")`

答案为 2



# 6 zipmap

zipmap constructs a map from a list of keys and a corresponding list of values. A map is denoted by { } whereas a list is donated by `[ ]`.

> `zipmap(["a", "b"], [1, 2])`

结果是 

```
{ "a" = 1 "b" = 2 }


不是 [ "a" = 1 "b" = 2 ]
```
