
# 1 lookup

https://developer.hashicorp.com/terraform/language/functions/lookup

lookup retrieves the value of a single element from a map, given its key. If the given key does not exist, the given default value is returned instead.

lookup(map, key, default)

For historical reasons, the `default` parameter is actually optional. However, omitting `default` is deprecated since v0.7 because that would then be equivalent to the native index syntax, `map[key]`.

```
> lookup({a="ay", b="bee"}, "a", "what?")
ay
> lookup({a="ay", b="bee"}, "c", "what?")
what?
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




