# Java Core Concepts - Interview Study Guide

**Created:** Thursday, February 06, 2026

---

## **OOPs Concepts (Inheritance, Polymorphism, Encapsulation, Abstraction)**

### **INHERITANCE** - Child class gets properties/methods from parent
```java
class Animal {
    void eat() { System.out.println("Eating"); }
}
class Dog extends Animal {  // Dog inherits eat()
    void bark() { System.out.println("Barking"); }
}
```

### **POLYMORPHISM** - One interface, many implementations
```java
Animal a = new Dog();  // Parent reference, child object
a.sound();  // Calls Dog's version of sound()
```

### **ENCAPSULATION** - Hide data, provide controlled access
```java
class Account {
    private double balance;  // Hidden
    public double getBalance() { return balance; }  // Public getter
}
```

### **ABSTRACTION** - Hide complex details, show only essentials
```java
abstract class Shape {
    abstract void draw();  // Must be implemented by child
}
```

---

## **Collections Framework (List, Set, Map, Queue and their implementations)**

### **LIST** - Ordered collection, allows duplicates
```java
List<String> arrayList = new ArrayList<>();  // Fast random access
List<String> linkedList = new LinkedList<>();  // Fast insert/delete

arrayList.add("Apple");
arrayList.get(0);  // Access by index
```

### **SET** - No duplicates allowed
```java
Set<String> hashSet = new HashSet<>();  // No order, O(1) lookup
Set<String> linkedHashSet = new LinkedHashSet<>();  // Insertion order
Set<String> treeSet = new TreeSet<>();  // Sorted order

hashSet.add("Apple");
hashSet.add("Apple");  // Ignored - no duplicates
```

### **MAP** - Key-value pairs, keys are unique
```java
Map<String, Integer> hashMap = new HashMap<>();  // No order
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();  // Insertion order
Map<String, Integer> treeMap = new TreeMap<>();  // Sorted by keys

hashMap.put("Alice", 25);
hashMap.get("Alice");  // Returns 25
```

### **QUEUE** - First-In-First-Out (FIFO)
```java
Queue<String> queue = new LinkedList<>();
queue.offer("First");  // Add to queue
queue.poll();  // Remove and return first element
```

---

## **Generics**

### **Purpose:** Type safety at compile time
```java
// Without generics - runtime error risk
List list = new ArrayList();
list.add("String");
Integer num = (Integer) list.get(0);  // ClassCastException at runtime!

// With generics - compile time safety
List<String> list = new ArrayList<>();
list.add("String");
// list.add(123);  // Compile error - prevented!

// Generic class
class Box<T> {
    private T item;
    public void set(T item) { this.item = item; }
    public T get() { return item; }
}

Box<Integer> intBox = new Box<>();
Box<String> strBox = new Box<>();
```

---

## **Exception Handling**

### **Checked exceptions** - Must handle at compile time
```java
try {
    FileReader file = new FileReader("file.txt");  // Checked exception
} catch (FileNotFoundException e) {
    System.out.println("File not found");
} finally {
    // Always executes - cleanup code here
}
```

### **Unchecked exceptions** - Runtime exceptions
```java
// NullPointerException, ArrayIndexOutOfBoundsException
String str = null;
str.length();  // Throws NullPointerException at runtime
```

### **Try-with-resources** - Auto-close resources
```java
try (FileReader fr = new FileReader("file.txt")) {
    // Use resource
}  // Automatically closes, even if exception occurs
```

---

## **Java 8+ Features (Lambda, Streams, Optional, Method references)**

### **LAMBDA** - Anonymous function, shorthand for functional interface
```java
// Old way
Runnable r = new Runnable() {
    public void run() { System.out.println("Running"); }
};

// Lambda way
Runnable r = () -> System.out.println("Running");

// With parameters
Comparator<Integer> comp = (a, b) -> a.compareTo(b);
```

### **STREAMS** - Process collections functionally
```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

names.stream()
     .filter(name -> name.startsWith("J"))  // Filter
     .map(String::toUpperCase)              // Transform
     .forEach(System.out::println);         // Action
```

### **OPTIONAL** - Avoid null pointer exceptions
```java
Optional<String> optional = Optional.ofNullable(getName());

// Safe way to handle potential null
String name = optional.orElse("Unknown");  // Default value
optional.ifPresent(n -> System.out.println(n));  // Only if present
```

### **METHOD REFERENCES** - Shorthand for lambda
```java
// Lambda
names.forEach(name -> System.out.println(name));

// Method reference
names.forEach(System.out::println);

// Types:
Class::staticMethod
object::instanceMethod
Class::instanceMethod
Class::new  // Constructor reference
```

---

## **Equals/HashCode Contract**

### **Rule:** If two objects are equal (equals() returns true), they MUST have same hashCode
```java
class Person {
    String name;
    int age;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && name.equals(person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);  // Must use same fields as equals
    }
}

// Why? HashMap/HashSet rely on this contract
// If equals() says objects are same, hashCode() must agree
```

---

## **Immutability and Final Keyword**

### **FINAL keyword** - Cannot be changed after initialization
```java
final int x = 10;
// x = 20;  // Compile error

final class CannotExtend {}  // Cannot be inherited
// class Child extends CannotExtend {}  // Error

class Parent {
    final void cannotOverride() {}  // Cannot be overridden
}
```

### **IMMUTABLE class** - State cannot change after creation
```java
public final class ImmutablePerson {
    private final String name;
    private final int age;
    
    public ImmutablePerson(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    // No setters - cannot modify after creation
}

// Benefits: Thread-safe, cacheable, predictable
// Example: String class is immutable
```

---

## **String, StringBuilder, StringBuffer**

### **STRING** - Immutable, thread-safe
```java
String s = "Hello";
s = s + " World";  // Creates new String object, original unchanged
// Use when: Not modifying much, need thread safety
```

### **STRINGBUILDER** - Mutable, NOT thread-safe, faster
```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Modifies same object
sb.insert(5, ",");    // "Hello, World"
// Use when: Building strings in single thread, performance matters
```

### **STRINGBUFFER** - Mutable, thread-safe, slower
```java
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World");  // Thread-safe modification
// Use when: Building strings in multi-threaded environment
```

### **Simple rule:**
- String = Don't change often
- StringBuilder = Change often, single thread (most common)
- StringBuffer = Change often, multiple threads

---

## **Serialization/Deserialization**

### **Purpose:** Convert object to byte stream (save to file/send over network)
```java
// Serialization - Object to bytes
class Person implements Serializable {
    private String name;
    private transient String password;  // Won't be serialized
}

// Write object to file
try (ObjectOutputStream out = new ObjectOutputStream(
        new FileOutputStream("person.ser"))) {
    out.writeObject(person);
}

// Deserialization - Bytes back to object
try (ObjectInputStream in = new ObjectInputStream(
        new FileInputStream("person.ser"))) {
    Person person = (Person) in.readObject();
}

// Use transient for fields you don't want to save
// serialVersionUID ensures version compatibility
```

---

## **Reflection API**

### **Purpose:** Inspect and manipulate classes at runtime
```java
// Get class information
Class<?> clazz = Person.class;
String className = clazz.getName();

// Get methods
Method[] methods = clazz.getDeclaredMethods();

// Get fields
Field[] fields = clazz.getDeclaredFields();

// Create instance dynamically
Object obj = clazz.getDeclaredConstructor().newInstance();

// Access private field
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);  // Bypass private
field.set(obj, "John");

// Invoke method
Method method = clazz.getMethod("getName");
String name = (String) method.invoke(obj);

// Use cases: Frameworks (Spring, Hibernate), testing, serialization
// Warning: Slow, breaks encapsulation - use sparingly
```

---

## **Annotations and Custom Annotations**

### **Built-in annotations**
```java
@Override  // Ensures method overrides parent method
@Deprecated  // Marks as obsolete
@SuppressWarnings("unchecked")  // Suppresses compiler warnings
@FunctionalInterface  // Ensures interface has single abstract method
```

### **Custom annotation**
```java
// Define annotation
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
@Target(ElementType.METHOD)  // Can be applied to methods
public @interface MyAnnotation {
    String value() default "";
    int count() default 0;
}

// Use annotation
@MyAnnotation(value = "test", count = 5)
public void myMethod() {}

// Process annotation at runtime
Method method = obj.getClass().getMethod("myMethod");
if (method.isAnnotationPresent(MyAnnotation.class)) {
    MyAnnotation ann = method.getAnnotation(MyAnnotation.class);
    System.out.println(ann.value());  // "test"
}
```

---

## **Garbage Collection Types (G1GC, ZGC, etc.)**

### **Purpose:** Automatically reclaim memory from unused objects

### **Serial GC** - Single thread, small apps
```bash
-XX:+UseSerialGC
# Good for: Small heap, single CPU
```

### **Parallel GC** - Multiple threads, throughput
```bash
-XX:+UseParallelGC
# Good for: Batch processing, multi-core systems
```

### **G1GC (Garbage First)** - Default since Java 9, balanced
```bash
-XX:+UseG1GC
# Good for: Large heaps, predictable pause times
# Divides heap into regions, collects regions with most garbage first
```

### **ZGC** - Ultra-low latency
```bash
-XX:+UseZGC
# Good for: Large heaps (multi-TB), pause times < 10ms
# Scales to huge heaps with minimal pause impact
```

### **Shenandoah GC** - Low pause times
```bash
-XX:+UseShenandoahGC
# Good for: Low latency requirements
# Concurrent garbage collection
```

### **Simple comparison:**
- Need low memory/single CPU? → Serial GC
- Need high throughput? → Parallel GC
- Balanced needs, large heap? → G1GC (default choice)
- Need ultra-low latency? → ZGC

---

## **JVM Architecture and Memory Areas (Heap, Stack, Metaspace)**

### **HEAP** - Objects live here
```java
Person p = new Person();  // Person object created in heap
// Shared among all threads
// Garbage collected
```

### **STACK** - Method calls and local variables
```java
void method() {
    int x = 10;  // x stored in stack
    Person p;    // Reference stored in stack, object in heap
}
// Each thread has its own stack
// Automatically cleaned when method returns
```

### **METASPACE** - Class metadata (replaced PermGen in Java 8)
```java
// Stores: Class definitions, method data, constant pool
// Grows dynamically (unlike old PermGen)
// -XX:MaxMetaspaceSize=512m  // Set max size
```

### **METHOD AREA** - Part of heap, stores class-level data
```java
// Static variables, constants, method code
```

### **PC REGISTER** - Current instruction address for each thread

### **NATIVE METHOD STACK** - Native method calls (C/C++)

### **Simple memory model:**
```
┌─────────────────────┐
│   HEAP (Objects)    │ ← Shared, GC'd
├─────────────────────┤
│ METASPACE (Classes) │ ← Shared, class metadata
└─────────────────────┘

Per Thread:
┌──────────────┐
│ STACK        │ ← Local vars, method calls
├──────────────┤
│ PC Register  │ ← Instruction pointer
└──────────────┘
```

---

## **ClassLoaders**

### **Purpose:** Load classes into JVM at runtime

### **Three types:**
```java
// 1. Bootstrap ClassLoader (native, loads core Java classes)
//    Loads: rt.jar, java.lang.*, etc.

// 2. Extension ClassLoader (loads jre/lib/ext)
//    Loads: javax.*, extensions

// 3. Application ClassLoader (loads CLASSPATH)
//    Loads: Your application classes

// Hierarchy: Bootstrap → Extension → Application
```

### **How it works:**
```java
// Delegation model: child asks parent first
// 1. Application ClassLoader asked to load class
// 2. Delegates to Extension ClassLoader
// 3. Extension delegates to Bootstrap ClassLoader
// 4. If Bootstrap can't find, Extension tries
// 5. If Extension can't find, Application tries

// Custom ClassLoader
class MyClassLoader extends ClassLoader {
    @Override
    public Class<?> findClass(String name) {
        byte[] classData = loadClassData(name);
        return defineClass(name, classData, 0, classData.length);
    }
}
```

---

## **Enum Usage and Patterns**

### **Basic enum**
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

Day day = Day.MONDAY;

// Enum with values() and valueOf()
for (Day d : Day.values()) {  // Get all enum constants
    System.out.println(d);
}
Day day = Day.valueOf("MONDAY");  // String to enum
```

### **Enum with fields and methods**
```java
enum Size {
    SMALL(10), MEDIUM(20), LARGE(30);
    
    private final int value;
    
    Size(int value) { this.value = value; }
    
    public int getValue() { return value; }
}

Size s = Size.MEDIUM;
System.out.println(s.getValue());  // 20
```

### **Enum for Strategy Pattern**
```java
enum Operation {
    ADD {
        public int apply(int a, int b) { return a + b; }
    },
    SUBTRACT {
        public int apply(int a, int b) { return a - b; }
    };
    
    public abstract int apply(int a, int b);
}

int result = Operation.ADD.apply(5, 3);  // 8
```

### **Benefits:**
- Type-safe constants
- Cannot create new instances
- Built-in methods (values(), valueOf(), ordinal())
- Can use in switch statements

---

## **Static vs Instance Members**

### **STATIC** - Belongs to class, shared by all instances
```java
class Counter {
    static int count = 0;  // Shared by all Counter objects
    
    static void increment() {  // Can call without object
        count++;
    }
}

Counter.increment();  // Call on class, not object
System.out.println(Counter.count);  // Access on class
```

### **INSTANCE** - Belongs to object, each object has its own
```java
class Person {
    String name;  // Each Person has their own name
    
    void sayHello() {  // Must create object to call
        System.out.println("Hello, " + name);
    }
}

Person p1 = new Person();
p1.name = "Alice";
p1.sayHello();  // Must call on object
```

### **Key differences:**
```java
class Example {
    static int staticVar = 10;    // One copy for all
    int instanceVar = 20;         // Each object has its own
    
    static void staticMethod() {
        // Can access: static members only
        // Cannot access: instanceVar, instanceMethod()
    }
    
    void instanceMethod() {
        // Can access: both static and instance members
        System.out.println(staticVar);    // OK
        System.out.println(instanceVar);  // OK
    }
}
```

### **Memory:**
- Static: Method area (shared)
- Instance: Heap (per object)

---

## **Nested and Inner Classes**

### **STATIC NESTED CLASS** - Can exist without outer class instance
```java
class Outer {
    static class StaticNested {
        void display() {
            System.out.println("Static nested");
        }
    }
}

Outer.StaticNested nested = new Outer.StaticNested();  // No outer needed
nested.display();
```

### **INNER CLASS** - Needs outer class instance
```java
class Outer {
    private int x = 10;
    
    class Inner {
        void display() {
            System.out.println(x);  // Can access outer's private members
        }
    }
}

Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();  // Must use outer instance
inner.display();
```

### **LOCAL CLASS** - Defined inside method
```java
void method() {
    final int x = 10;
    
    class Local {
        void display() {
            System.out.println(x);  // Can access final local variables
        }
    }
    
    Local local = new Local();
    local.display();
}
```

### **ANONYMOUS CLASS** - No name, used once
```java
Runnable r = new Runnable() {  // Anonymous inner class
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

### **When to use:**
- Static nested: Logically related, doesn't need outer instance
- Inner: Needs access to outer's instance members
- Local: Helper class needed only in one method
- Anonymous: One-time use, simple implementation

---

## **Functional Interfaces**

### **Definition:** Interface with exactly ONE abstract method
```java
@FunctionalInterface  // Optional but recommended
interface Calculator {
    int calculate(int a, int b);  // Single abstract method
    
    // Can have default and static methods
    default void print() { }
    static void info() { }
}

// Use with lambda
Calculator add = (a, b) -> a + b;
int result = add.calculate(5, 3);  // 8
```

### **Built-in functional interfaces:**
```java
// Predicate<T> - Takes T, returns boolean
Predicate<String> isEmpty = s -> s.isEmpty();

// Function<T, R> - Takes T, returns R
Function<String, Integer> length = s -> s.length();

// Consumer<T> - Takes T, returns void
Consumer<String> print = s -> System.out.println(s);

// Supplier<T> - Takes nothing, returns T
Supplier<String> supplier = () -> "Hello";

// BiFunction<T, U, R> - Takes T and U, returns R
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
```

---

## **Record Classes (Java 14+)**

### **Purpose:** Immutable data carriers (like DTOs)
```java
// Old way
class PersonOld {
    private final String name;
    private final int age;
    
    public PersonOld(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String name() { return name; }
    public int age() { return age; }
    
    // Plus equals(), hashCode(), toString()...
}

// Record way - all above auto-generated!
record Person(String name, int age) {}

Person p = new Person("Alice", 25);
System.out.println(p.name());  // Alice
System.out.println(p);  // Person[name=Alice, age=25]

// Automatic: constructor, getters, equals(), hashCode(), toString()
```

### **Custom methods in records:**
```java
record Person(String name, int age) {
    // Compact constructor - validation
    public Person {
        if (age < 0) throw new IllegalArgumentException();
    }
    
    // Custom method
    public boolean isAdult() {
        return age >= 18;
    }
}
```

### **Benefits:** Less boilerplate, immutable by default, clear intent

---

## **Sealed Classes (Java 17+)**

### **Purpose:** Control which classes can extend/implement
```java
// Only Dog, Cat, Bird can extend Animal
public sealed class Animal permits Dog, Cat, Bird {
    // Common code
}

final class Dog extends Animal {}  // Must be final, sealed, or non-sealed
final class Cat extends Animal {}
non-sealed class Bird extends Animal {}  // Others can extend Bird

// Compiler error:
// class Fish extends Animal {}  // Not permitted!
```

### **Why use:**
```java
// Exhaustive switch (compiler knows all subtypes)
public String makeSound(Animal animal) {
    return switch(animal) {
        case Dog d -> "Woof";
        case Cat c -> "Meow";
        case Bird b -> "Tweet";
        // No default needed - compiler knows all cases
    };
}
```

### **Benefits:**
- Limited inheritance hierarchy
- Pattern matching knows all subtypes
- More maintainable domain models

---

## **Pattern Matching for Switch (Java 17+)**

### **Old way:**
```java
Object obj = "Hello";

if (obj instanceof String) {
    String s = (String) obj;  // Cast needed
    System.out.println(s.length());
}
```

### **Pattern matching (Java 16+):**
```java
if (obj instanceof String s) {  // Pattern variable 's'
    System.out.println(s.length());  // No cast needed
}
```

### **Switch with pattern matching (Java 17+):**
```java
static String format(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l -> String.format("long %d", l);
        case String s -> String.format("String %s", s);
        case null -> "null";
        default -> obj.toString();
    };
}
```

### **With guards:**
```java
static String test(Object obj) {
    return switch (obj) {
        case String s && s.length() > 5 -> "Long string";  // Guard
        case String s -> "Short string";
        default -> "Not a string";
    };
}
```

---

## **Text Blocks (Java 15+)**

### **Old way (multi-line strings):**
```java
String html = "<html>\n" +
              "  <body>\n" +
              "    <p>Hello</p>\n" +
              "  </body>\n" +
              "</html>";
```

### **Text blocks way:**
```java
String html = """
              <html>
                <body>
                  <p>Hello</p>
                </body>
              </html>
              """;
```

### **JSON example:**
```java
String json = """
              {
                "name": "John",
                "age": 30,
                "city": "New York"
              }
              """;
```

### **SQL example:**
```java
String query = """
               SELECT id, name, email
               FROM users
               WHERE age > 18
               ORDER BY name
               """;
```

### **Benefits:**
- More readable
- No escape characters needed
- Preserves formatting
- Great for SQL, JSON, HTML, XML

---

## **Quick Reference Summary**

### **Collections Quick Pick:**
- Need order + duplicates? → ArrayList
- Need fast insert/delete? → LinkedList
- Need unique elements? → HashSet
- Need unique + order? → LinkedHashSet
- Need sorted? → TreeSet
- Need key-value? → HashMap

### **String Manipulation:**
- Immutable/rarely change? → String
- Mutable/single thread? → StringBuilder
- Mutable/multi-thread? → StringBuffer

### **Memory:**
- Objects → Heap
- Local variables → Stack
- Class metadata → Metaspace
- Static variables → Method Area

### **GC Choice:**
- Small app? → Serial GC
- Throughput? → Parallel GC
- Balanced/default? → G1GC
- Ultra-low latency? → ZGC

### **Modern Java Features (14+):**
- Data classes → Records
- Controlled inheritance → Sealed classes
- Better switch → Pattern matching
- Multi-line strings → Text blocks

---

**End of Study Guide**