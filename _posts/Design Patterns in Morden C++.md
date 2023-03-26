# Design Patterns in Modern C++
Reusable Approaches for Object-Oriented Software Design


## CHAPTER 1: Introduction

### 1. Important Concepts
* Curiously Recurring Template Pattern

Idea: an inheritor passes *itself* as a template argument to its base class.
```cpp
struct Foo : Somebase<Foo>{
    ...
}
```

* Mixin Inheritance
```cpp
template <typename T> struct Mixin : T {
    ...
}
```

* Properties
```cpp
class Person{
    int age_;
public:
    int get_age() const {
        return age_;
    }
    void set_age(int value) const {
        age_ = value;
    }
    __declspec(property(get=get_age, put=set_age)) int age;
}
```

### 2. The SOLID Design Principles
* Single Responsibility Principle (SRP)
    1. Each class has only one responsibility, and therefore has only one reason to change.

* Open-Closed Principle (OCP)
    1. A type is open for extension but closed for modification.

* Liskov Substitution Principle (LSP)
    1. If an interface takes an object of type Parent, it should equally take an object of type Child without anything breaking.
* Interface Segregation Principle (ISP)
    1. Segregate parts of a complicated interface into separate interfaces so as to avoid forcing implementors to implement functionality that they do not really need.
* Dependency Inversion Principle (DIP)
    1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
    2. Abstractions should not depend on details. Details should depend on abstractions.

## CHAPTER 2: Builder
### Key characteristics of a Builder:
* Builders can have a fluent interface that is usable for complicated construction using a single invocation chain. To support this, builder functions should return this or *this.
* To force the user of the API to use a Builder, we can make the target object’s constructors inaccessible and then define a static create() function that returns the builder.
* A builder can be coerced to the object itself by defining the appropriate operator.
* Groovy-style builders are possible in C++ thanks
to uniform initializer syntax. This approach is very general, and allows for the creation of diverse DSLs.
* A single builder interface can expose multiple subbuilders. Through clever use of inheritance and fluent interfaces, one can jump from one builder to another with ease.

> Note: simple objects that are unambiguously constructed from a limited number of sensibly named constructor parameters should probably use a constructor (or dependency injection) without necessitating a Builder as such.

## CHAPTER 3: Factories

### 1. Factory Method
```cpp
struct Point {
private:
    float x, y;

protected:
    Point(const float x, const float y)
            : x{x}, y{y} {}

public:
    static Point NewCartesian(float x, float y) {
        return {x, y};
    }


    static Point NewPolar(float r, float theta) {
        return {r * cos(theta), r * sin(theta)};
    }
    // other members here
};
```
Each of the above static functions is called a *Factory Method*.  
To create a point, simply write:
```cpp
auto p = Point::NewPolar(5, M_PI_4);
```

### 2. Factory
```cpp
struct Point {
    float x, y;

    Point(float x, float y) : x(x), y(y) {}
};

struct PointFactory {
    static Point NewCartesian(float x, float y) {
        return Point{x, y};
    }

    static Point NewPolar(float r, float theta) {
        return Point{r * cos(theta), r * sin(theta)};
    }
};
```
Write as follows:
```cpp
auto my_point = PointFactory::NewCartesian(3, 4);
```

### 3. Inner Factory
```cpp
struct Point {
private:
    Point(float x, float y) : x(x), y(y) {}

    struct PointFactory {
    private:
        PointFactory() {}

    public:
        static Point NewCartesian(float x, float y) {
            return {x, y};
        }

        static Point NewPolar(float r, float theta) {
            return {r * cos(theta), r * sin(theta)};
        }
    };

public:
    float x, y;
    static PointFactory Factory;
};
```
Use the factory as:
```cpp
auto pp = Point::Factory.NewCartesian(2, 3);
```

### 4. Abstract Factory
```cpp
struct HotDrink {
    virtual void prepare(int volume) = 0;
};

struct Tea : HotDrink {

    void prepare(int volume) override {
        cout << "Take tea bag, boil water, pour " << volume << "ml." << endl;
    }
};

struct Coffee : HotDrink {

    void prepare(int volume) override {
        cout << "Take coffee bag, boil water, pour " << volume << "ml." << endl;
    }
};

struct HotDrinkFactory {
    virtual unique_ptr<HotDrink> make() const = 0;
};

struct TeaFactory : HotDrinkFactory {
    unique_ptr<HotDrink> make() const override {
        return make_unique<Tea>();
    }
};

struct CoffeeFactory : HotDrinkFactory {
    unique_ptr<HotDrink> make() const override {
        return make_unique<Coffee>();
    }
};

class DrinkFactory {
    map<string, unique_ptr<HotDrinkFactory>> hot_factories;
public:
    DrinkFactory() {
        hot_factories["coffee"] = make_unique<CoffeeFactory>();
        hot_factories["tea"] = make_unique<TeaFactory>();
    }

    unique_ptr<HotDrink> make_drink(const string &name) {
        auto drink = hot_factories[name]->make();
        drink->prepare(200); // oops!
        return drink;
    }
};
```
Use the factory as:
```cpp
auto drink_factory = DrinkFactory();
auto coffee = drink_factory.make_drink("coffee");
```

### 5. Functional Factory
```cpp
class DrinkWithVolumeFactory {
    map<string, function<unique_ptr<HotDrink>()>> factories;
public:
    DrinkWithVolumeFactory() {
        factories["tea"] = [] {
            auto tea = make_unique<Tea>();
            tea->prepare(200);
            return tea;
        }; // similar for Coffee
    }

    unique_ptr<HotDrink> make_drink(const string &name) {
        return factories[name]();
    }

};
```
Use the factory as:
```cpp
auto drink_factory = DrinkWithVolumeFactory();
auto tea = drink_factory.make_drink("tea");
```

### 6. Summary
1. terminology
* A *factory method* is a class member that acts as a way of creating object. It typically replaces a constructor.
* A *factory* is typically a separate class that knows how to construct objects, though if you pass a function (as in std::function or similar) that constructs objects, this argument is also called a factory.
* An *abstract factory* is, as its name suggests, an abstract class that can be inherited by concrete classes that offer a family of types. Abstract factories are rare in the wild.
2. A factory has several critical advantages over a constructor call, namely:
* A factory can say no, meaning that instead of actually returning an object it can return, for example, a nullptr.
* Naming is better and unconstrained, unlike constructor name.
* A single factory can make objects of many different types.
* A factory can exhibit polymorphic behavior, instantiating a class and returning it through its base class’ reference or pointer.
* A factory can implement caching and other storage optimizations; it is also a natural choice for approaches such as pooling or the Singleton pattern.

> Note: Factory is different from Builder in that, with a Factory, you typically create an object in one go, whereas with Builder, you construct the object piecewise by providing information in parts.

## CHAPTER 4: Prototype
* Prototype: A model object that we can make copies of, customize those copies, and then use them.
* The Prototype design pattern embodies the notion of deep copying of objects so that, instead of doing full initialization each time, you can take a premade object, copy it, fiddle it a little bit, and then use it independently of the original.

### The only two ways of implementing the Prototype pattern in C++
* Writing code that correctly duplicates your object, that is, performs a deep copy. This can be done in a copy constructor/copy assignment operator or in a separate member function.
* Write code for the support of serialization/ deserialization and then use this mechanism to implement cloning as serialization immediately followed by deserialization. This carries the extra computational cost; its significance depends on how often you need to do the copying. The *only* advantage of this approach, compared with using, say, copy constructors, is that you get serialization for free.

## CHAPTER 5: Singleton
### Classic Implementation
```cpp
struct Database {
protected:
    Database() {
        /* do what you need to do */
    }

public:
    static Database &get() {
        // thread-safe in C++11
        static Database database;
        return database;
    }

    Database(Database const &) = delete;

    Database(Database &&) = delete;

    Database &operator=(Database const &) = delete;

    Database &operator=(Database &&) = delete;
};
```
### A particularly nasty trick
You can impletment get() as a heap allocation (so that only the pointer, not the entire object, is static).
```cpp
static Database &get() {
    static Database *database = new Database();
    return *database;
}
```

## CHAPTER 6: Adapter
Adapter allows you to adapt the interface you have to the interface you need.
### The only issue with adapters 
In the process of adaptation, you sometimes end up generating temporary data so as to satisfy some other representation of data. And when this happens, turn to caching: 
* Ensure that new data is only generated when necessary. 
* Clean up stale data when the cached objects have changed.

## CHAPTER 7: Bridge
### The Pimpl Idiom
```cpp
struct Person {
    std::string name;

    void greet(Person *p) {
        impl->greet(this);
    }

    Person() : impl(new PersonImpl) {}

    ~Person() { delete impl; }

    struct PersonImpl {
        void greet(Person *p) {
            printf("hello %s", p->name.c_str());
        }
    };

    PersonImpl *impl; // good place for gsl::owner<T>
};
```
Three advantages:
* A larger proportion of the class implementation is actually hidden. If your Person class required a rich API full of private/protected members, you’d be exposing all those details to your clients, even if they could never access those members due to private/protected access modifiers. With Pimpl, they can only be given the public interface.
* Modifying the data members of the hidden Impl class does not affect binary compatibility.
* The header file only needs to include the header files needed for the declaration, not the implementation. For example, if Person requires a private member of type `vector<string>`, you would be forced to #include both `<vector>` and `<string>` in the Person.h header (and this is transitive, so anyone using Person.h would be including them too). With the Pimpl idiom, this can be done in the .cpp file instead.

A side effect: reduce compilation speed.

### Bridge
```cpp
struct Renderer {
    virtual void render_circle(float x, float y, float radius) = 0;
};

struct VectorRenderer : Renderer {
    void render_circle(float x, float y, float radius) override {
        cout << "Rasterizing circle of radius " << radius << endl;
    }
};

struct RasterRenderer : Renderer {
    void render_circle(float x, float y, float radius) override {
        cout << "Drawing a vector circle of radius " << radius << endl;
    }
};

struct Shape {
protected:
    Renderer &renderer;

    Shape(Renderer &renderer) : renderer{renderer} {}

public:
    virtual void draw() = 0;

    virtual void resize(float factor) = 0;
};

struct Circle : Shape {
    float x, y, radius;

    void draw() override {
        renderer.render_circle(x, y, radius);
    }

    void resize(float factor) override {
        radius *= factor;
    }

    Circle(Renderer &renderer, float x, float y, float radius) : Shape{renderer}, x{x}, y{y}, radius{radius} {}
};
```
*Bridge* here is a Renderer, for example:
```cpp
RasterRenderer rr;
Circle raster_circle{rr, 5, 5, 5};
raster_circle.draw();
raster_circle.resize(2);
raster_circle.draw();
```

> Note: The Bridge serving as a connector or glue, connecting two pieces together. The use of abstraction (interfaces) allows components to interact with one another without really being aware of the concrete implementations. That said, the participants of the Bridge pattern do need to be aware of each other’s existence.

## CHAPTER 8: Composite
The Composite design pattern allows us to provide identical interfaces for individual objects and collections of objects.

### Array Backed Properties
```cpp
class Creature {
    enum Abilities {
        str, agl, intl, count
    };
    array<int, count> abilities;

    int get_strength() const { return abilities[str]; }

    void set_strength(int value) { abilities[str] = value; }

    // same for other properties
};
```
This makes calculations such as sum(), average(), and max() become truly trivial.

### Grouping Graphic Objects
```cpp
struct GraphicObject {
    virtual void draw() = 0;
};

struct Circle : GraphicObject {
    void draw() override {
        std::cout << "Circle" << std::endl;
    }
};

struct Group : GraphicObject {
    std::string name;

    explicit Group(const std::string &name)
            : name{name} {}

    void draw() override {
        std::cout << "Group " << name.c_str() << " contains:" << std::endl;
        for (auto &&o : objects) {
            o->draw();
        }
    }

    std::vector<GraphicObject *> objects;
};
```
Here’s how this API can be used:
```cpp
Group root("root");
Circle c1, c2;
root.objects.push_back(&c1);
Group subgroup("sub");
subgroup.objects.push_back(&c2);
root.objects.push_back(&subgroup);
root.draw();
```

### Neural Networks
```cpp
template<typename Self>
struct SomeNeurons {
    template<typename T>
    void connect_to(T &other) {
        for (auto &from : *static_cast<Self *>(this)) {
            for (auto &to : other) {
                from.out.push_back(&to);
                to.in.push_back(&from);
            }
        }
    }
};
```
> Note: the *Neural Networks* case is a much special case in my opinion... It seems hard to find a scenario perfectly suit this case.

## CHAPTER 9: Decorator
The Decorator pattern allows us to enhance existing types without either modifying the original types (Open-Closed Principle) or causing an explosion of the number of derived types.
* Dynamic composition allows you to compose something at runtime, typically by passing around references. It allows maximum flexibility, since the composition can happen at runtime in response to, for example, the user’s input.
* Static composition implies that the object and its enhancements are composed at compile time via the use of templates. This means the exact set of enhancements on an object needs to be known at the moment of compilation, since it cannot be modified later.

A decorator gives a class additional functionality while adhering to the OCP. Its crucial aspect is composability: several decorators can be applied to an object in any order.
* Dynamic decorators can store references (or even store the entire values, if you want!) of the decorated objects and provide dynamic (runtime) composability, at the expense of not being able to access the underlying objects’ own members.
* Static decorators use mixin inheritance (inheriting from template parameter) to compose decorators at compile-time. This loses any sort of runtime flexibility (you cannot recompose objects) but gives you access to the underlying object’s members. These objects are also fully initializable through constructor forwarding.
* Functional decorators can wrap either blocks of code or particular functions to allow composition of behaviors.

## CHAPTER 10: Façade
The Façade design pattern is a way of putting a simple interface in front of one or more complicated subsystems.

## CHAPTER 11: Flyweight
* A Flyweight (also sometimes called a token or a cookie) is a temporary component that acts as a “smart reference” to something.
* Typically, flyweights are used in situations where you have a very large number of very similar objects, and you want to minimize the amount of memory that is dedicated to storing all these values.
