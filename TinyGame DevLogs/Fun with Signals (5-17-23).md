## Bind your Methods!
Today I spent about an hour trying to figure out why Godot could not find a method in my `GameManager` class when trying to reference it using the `input->connect()` method. This method takes in a `Callable(godot::Object, string)` object which takes in a godot object and a string representing a callable method that is a member of that object.

My first implementation was to rather crudely just reference my `GameManager` class using the `this` keyword and the name of the method (`_on_joy_input_changed`) I wanted triggered everytime the signal `joy_connected_changed` was sent out by the `Input` class.

The problem is that godot had no reference to this method because I forgot to use the `ClassDB::bind_method` function within `GameManager`'s `_bind_methods` method (jesus) and thus the engine itself had no reference to it. This would have also applied if I had made a custom signal and used it in place of `joy_connection_changed`. I would have to define it using the `ClassDB:add_signal` method (or just by using the `ADD_SIGNAL` macro) as demonstrated in the [GDExtension Example](https://docs.godotengine.org/en/stable/tutorials/scripting/gdextension/gdextension_cpp_example.html).

### Bonus C++ inheritance fuckery.
During this process I had assumed that my problem was simply my use of the `this` keyword to reference my `GameManager` class due to the fact that when I would print the output of `this` it would return a pointer to the *base* class that `GameManager` is derived from (that being `Node3D`).
> Note that typically printing the `this` keyword using `std::cout` or something similar would just print the address in memory that `this` is pointing to, however using godot's `UtilityFunctions::print` method seems to format it a different way, including the name of the object followed by some kind of identifier.

So during this process I learned that the `this` is a keyword processed at compiler-time that will always point to the *base class* rather than the derived class. You can also use static binding `(Derived)this->method` or, of course, *overrides* to change what is actually returned when refering to a specific member of the object at *this* address (if that makes any sense)). The only confusion here is if I'm trying to reference my *derived* class as an address in memory, how do I differenciate it from that of my *base* class? I'm not entirely sure at this moment.

Further reading on this can be found [here](https://medium.com/geekculture/c-inheritance-memory-model-eac9eb9c56b5).
