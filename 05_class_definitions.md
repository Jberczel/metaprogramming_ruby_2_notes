# Chapter 5 - Class Definitions

Per author, this is most advanced chapter in entire book.

## Class Definitions Demystified

Usually you think of only defining methods within a class defintion, but you can put any code you want inside:

```ruby
class MyClass
  puts "hello"
end

# => Hello
```


Class definitions also return the value of the last statement, just like methods and blocks:

```ruby
result = class MyClass
  self
end


result # => MyClass
```

Keep in mind that Classes and Modules are objects, so they can be _self_ in this case. Concept will be important later in this chapter.

### The Current Class

Wherever you are in a Ruby program, you alway have current object: self.  You will also have a current class (or module).  You will also have a current class (or module).

You always have a reference to the current object (self), but how do you find current class?

- At top level, the current class is _Object_.
- In a method, the current class is the class of the current object.
- When you open a class with the _class_ keyword, that class becomes the current class. 


