## Definitions
### Encapsulation
*Encapsulation* means that an object presents only a limited amount of information to the outside world; the object's internal state and implementation details are hidden in order to simplify life for the user of the class because they need only to understand the class' limited interface rather than the intricate details of it's implementation.

### Inheritance
*Inheritance* allows new classes to be defined as *extensions* to prexisting classes.

### Polymorphism
*Polymorphism* is a language feature that allows a collection of objects of different types to be manipulated through a single *common interface*. 
```cpp
struct Shape {
	virtual void Draw() = 0; // pure virtual function
	virtual ~Shape() {} // ensure derived deconstructors are virtual
};

struct Circle : public Shape {
	virtual void Draw() {
		// Draw a circle
	}
};

struct Rectangle : public Shape {
	virtual void Draw() {
		// Draw a rectangle
	}
}

void drawShapes(std::list<Shape*>& shapes) {
	std::list<Shape*>::iterator pShape = shapes.begin();
	std::list<Shape*>::iterator pEnd = shapes.end();

	for (; pShape != pEnd; pShape++) {
		pShape()->draw(); // calls virtual function
	}
}
```

### Composition and Aggregation
*Composition* is the practice of using a *group* of *interacting* objects to accomplish a high-level task. Composition creates a "has-a" relationship between classes. *Aggregation* creates a "uses-a" relationship between classes.
> Note: Composition is typically underutilized compared to inheritance. This comes at the cost of code simplicity. E.g. A GUI window could be potentially derived from a class `Rectangle`, or it could *refer to* or *contain* a `Rectangle`, making the overall code simpler.

## Design Patterns

### Singleton
The *singleton* pattern ensures that a particular class has only one instance (the *singleton instance*) and provides a global point of access to it.

### Iterator
An *iterator* provides an efficient means of accessing the individual elements of a collection, without exposing the collection's underlying implementation. The iterator "knows" the implementation details of the collection so the user doesn't have to.

### Abstract Factory
An *abstract factory* provides an interface for creating families of related or dependent classes without specifying their concrete classes.

### Resource Acquisition is Initialization (RAII)
In the *RAII* pattern, resource acquisition and release are bound to the constructor and deconstructor of a class, respectively. When using this pattern, one would simply construct an instance of a class to acquire a resource, and then let it fall out of scope to release it automatically.
```cpp
/* When we need to allocate memory from a particular type of allocator, we push that allocator onto a global allocator stack. When we're done allocating, we pop that allocator off the stack. */
class AllocJanitor {
public:
explicit AllocJanitor(mem::Context context) {
	mem::PushAllocator(context);
}
~AllocJanitor() {
	mem:PopAllocator();
}
};

// Allocate temp buffers from single-frame allocator
{
AllocJanitor janitor(mem::Context::kSingleFrame);
U8* pByteBuffer = new U8[SIZE];
float* pFloatBuffer = new float[SIZE];
// Use buffers...
} // Janitor pops allocator when it falls out of scope
```
