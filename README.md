# elixir-playground

## Functions

- Lambda function

```elixir
add_one = fn x -> x + 1 end
Agent.get_and_update(bucket, fn v -> Map.pop(v, key) end)
```

- Capture operator

```elixir
add_one = &(&1+1)
Agent.get_and_update(bucket, &(Map.pop(&1, key)))
```

This is exactly the same as `lambda` version.

Capture operator can be mixed with operator `{}` or `[]`.

```elixir
return_list = &[&1, &2]
return_list.(1, 2) # [1, 2]

return_tuple = &{&1, &2}
return_tuple.(1, 2) # {1, 2}
```

- Nomral function

def function_name(argument_1, argument_2) do
   #code to be executed when function is called
end

- Inline form

`do foo() end` with `, do: foo`

```elixir
def my_func(args), do: "result"
# pattern matching form
def my_func(:ok), do: ...
def my_func(:error), do: ...
```

Either way, both forms are valid and you can use them interchangeably.

Read [here](https://elixirforum.com/t/is-end-keyword-optional-in-elixir-when-using-def/7882/6) for more details.

A do/end block is really just a keyword list under the hood, with a do key and some expression as the value. You can read a little more about it here 5. In other words

```elixir
do
  x + y
end
```

can also be represented as

```elixir
[do: x + y]
```

`def` and `defp` are just macros, so although they look a lot like keywords they are really just calls and therefore have the same characteristics as any other function/macro call with regards to syntax. They have an arity of two: the first argument is the call and the second is the expression. Another way to write a function definition could be: `def(add(x, y), [do: x + y])` When written this way it’s no longer ‘disguised’ as a keyword. It’s clear that it is just a macro call. Since parenthesis are optional for calls we can also use the form: `def add(x, y), [do: x + y]`, since the brackets on keyword lists are optional when used as the last parameter in a function call, we can also use this form: `def add(x, y), do: x + y`, now we’ve arrived at the ‘single-line’ form. Its clearer now why the comma and colon come into play. It isn’t a special syntax used for single line function definitions: the comma is just the parameter separator used in all function calls and the colon is for the key in the keyword list.

Finally, we can represent the do keyword list using the do/end block syntax and get:

```elixir
def add(x, y) do
  x + y
end
```

When using a `do/end` block as the the final argument in a function we can omit the leading comma (I’m not sure if I’ve ever seen this formally stated before, but it does seem to behave that way).

This same behavior applies for all other macros. For example, if is also a macro, so you can break it down the same way:

```elixir
if(something(), [do: foo()])

if something(), [do: foo()]

if something(), do: foo()

if something() do
  foo()
end
```

An `else` block (as well as try, catch, rescue, and a few others) is really just another key on the do keyword list, so in other words you can introduce the else block the same way:

```elixir
if(something(), [do: this(), else: that()])

if something(), [do: this(), else: that()]

if something(), do: this(), else: that()

if something() do
  this()
else
  that()
end
```

To summarize: def and defp are just macros calls, and as such follow the same syntax rules as other calls. Technically there aren’t two separate forms for function definitions, there are just varying degrees of sugar you can use to dress up function calls. That being said, you will encounter these two ‘forms’ enough in practice that it makes sense to distinguish them, so while there aren’t technically two forms I think most people will recognize them as being the two de-facto forms.

The single-line variant is used often, here are some sections from an elixir style-guide to give you an idea of when to use them vs the block variant: <https://github.com/christopheradams/elixir_style_guide#single-line-defs>

Edit: There is also another bit of sugar that I left off in these examples. Keyword lists are really lists of 2-value tuples in the form {atom key, mixed value}. So for example: `[do: foo(), else: bar()]` is really `[{:do, foo()}, {:else, bar()}]` Which makes the if example become in full:

```elixir
if(something(), [{:do, this()}, {:else, that()}])
if(something(), [do: this(), else: that()])
if something(), [do: this(), else: that()]
if something(), do: this(), else: that()
if something() do
  this()
else
  that()
end
```

It can be a little bit overwhelming and puzzling at first having to deal with these little syntactic intricacies. My first reaction to seeing the single line def form was “why is that comma there?! and why a colon too?!”. I thought having all these different layers of optional syntax was bizarre, and personally I would prefer there to be less ways to write something, not more.

I was initially turned off by it until I realized that most of what I considered to be language keywords were actually just macros. Then it started to make sense why some of these syntax choices were made (at least why I think they were made): they give you the ability to make macros that look identical to traditional keywords. This is why you can’t tell that def is really just a macro until you squint and peel back the layers.

Having this capability allows you to expand the language to add expressive, domain specific constructs. Phoenix has several great examples of this, like defining plug pipelines, route scopes, things like attaching plugs directly to a controller, etc. Ecto’s query DSL is another awesome example, same with ExUnit’s assertion and test definitions. Some of these constructs feel so natural and expressive that they feel like they are a part of the language itself, but they are actually more like libraries.
