# SOLID Principles in TypeScript

<h2 align="center">
  <img src="https://github.com/armanabkar/SOLID-Principles-TypeScript/blob/main/SOLID.png" alt="TypeScriptSOLID" width="550px" />
  <br>
</h2>

The SOLID principles were defined quite some time ago and are still relatable. Their goal is to make our software easier to understand, read, and extend. We accredit this concept to [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) from his paper from the year 2000. The actual SOLID acronym was defined later, though. In this article, we go through all of the SOLID principles and reflect them in TypeScript examples.

- S: [Single responsibility principle](#single-responsibility-principle)
- O: [Open-closed principle](#open-closed-principle)
- L: [Liskov substitution principle](#liskov-substitution-principle)
- I: [Interface segregation principle](#interface-segregation-principle)
- D: [Dependency inversion principle](#dependency-inversion-principle)

## Single responsibility principle
The single responsibility principle declares that a class should only be responsible for a single functionality. The above also refers to modules and functions.

    class Statistics {
      public computeSalesStatistics() {
        // ...
      }
      public generateReport() {
        // ...
      }
    }
We should split the above class into two separate ones. SOLID defines the concept of responsibility as a REASON TO CHANGE.

Robert C. Martin, [in his blogpost](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html "in his blogpost"), discusses how we can define a reason to change. He stresses that it shouldn’t be deliberated in terms of code. For example, refactoring is not a reason to change in terms of the single responsibility principle.

To define a reason to change, we need to investigate what is the responsibility of our program.

The Statistics class might be changed due to two different reasons:

- the logic of the sales statistic computation changes,
- the format of the report changes

The single responsibility principle highlights that the two above aspects put two different responsibilities on the Statistics class.

    class Statistics {
      public computeSalesStatistics() {
        // ...
      }
    }
	
    class ReportGenerator {
      public generateReport() {
        // ...
      }
    }
Now we have to separate classes, and each of them has a single reason to change, and therefore just one responsibility. Applying to the above principle makes our code easier to explain and understand.

## Open-closed principle
According to the open-closed principle, software entities should be open for extension but closed for modification.

The core idea of the above principle is that we should be able to add new functionalities without changing the existing code.
    
    class Rectangle {
      public width: number;
      public height: number;
      constructor(width: number, height: number) {
        this.width = width;
        this.height = height;
      }
    }
	
    class Circle {
      public radius: number;
      constructor(radius: number) {
        this.radius = radius;
      }
    }
Let’s say we want to create a function that calculates the area of an array of shapes. With our current design, it might look like that:

    function calculateAreasOfMultipleShapes(
      shapes: Array<Rectangle | Circle>
    ) {
      return shapes.reduce(
        (calculatedArea, shape) => {
          if (shape instanceof Rectangle) {
            return calculatedArea + shape.width * shape.height;
          }
          if (shape instanceof Circle) {
            return calculatedArea + shape.radius * Math.PI;
          }
        },
        0
      );
    }

The issue with the above approach is that when we introduce a new shape, we need to modify our  `calculateAreasOfMultipleShapes` function. This makes it open for modification and breaks the open-closed principle.

We can fix that by enforcing our shapes to have a method that returns the area.

    interface Shape {
      getArea(): number;
    }
	
    class Rectangle implements Shape {
      public width: number;
      public height: number;
      constructor(width: number, height: number) {
        this.width = width;
        this.height = height;
      }
      public getArea() {
        return this.width * this.height;
      }
    }
	
    class Circle implements Shape {
      public radius: number;
      constructor(radius: number) {
        this.radius = radius;
      }
      public getArea() {
        return this.radius * Math.PI;
      }
    }
Now that we are sure that all of our shapes have the  `getArea` function, we can use it further.

    function calculateAreasOfMultipleShapes(
      shapes: Shape[]
    ) {
      return shapes.reduce(
        (calculatedArea, shape) => {
          return calculatedArea + shape.getArea();
        },
        0
      );
    }
Now when we introduce a new shape, we don’t need to modify our  `calculateAreasOfMultipleShapes` function. We make it open for extension but closed for modification.

> We could achieve the above functionality also by using an [abstract class](https://www.typescriptlang.org/docs/handbook/classes.html#abstract-classes "abstract class") instead of an interface

## Liskov substitution principle
The above rule, introduced by [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov "Barbara Liskov"), also helps us ensure that changing one area of our system does not break other parts. To make this principle less confusing, we will break it down into multiple parts.

> Replacing an instance of a class with its child class should not produce any negative side effects

The first thing we notice in the above principle is that its main focus is class inheritance.

Let’s implement a straightforward and vivid example of how we can break the above principle.

    class Employee {
      protected permissions: any = new Set<string>();
     
      public hasPermission(permissionName: string) {
        return this.permissions.has(permissionName);
      }
      public addPermission(permissionName: string) {
        return this.permissions.add(permissionName);
      }
    }
	
    class Cashier extends Employee {
      protected permissions: string[] = [];
     
      public addPermission(permissionName: string) {
        this.permissions.push(permissionName);
      }
    }
	
    function isPersonAllowedToDeleteProducts(person: Employee) {
      return person.hasPermission('deleteProducts');
    }
The above code is very problematic because the implementation of  `permissions` differs in the  `Cashier` and the  `User`.

    const employee = new Employee();
    employee.addPermission('deleteProducts');
    isPersonAllowedToDeleteProducts(employee);
The above code works fine, but when we replace an instance of a parent class with its child class, we experience issues.

    const cashier = new Cashier();
    cashier.addPermission('deleteProducts');
    isPersonAllowedToDeleteProducts(cashier);
> TypeError: this.permissions.has is not a function

This situation is very vivid and shouldn’t happen in a properly typed TypeScript code. We had to use  `permissions: any` to allow the  `Cashier` to extend the  `User` improperly.

#### Validation conditions
A more refined example is with preconditions and validation.

    class Employee {
      protected permissions = new Set<string>();
     
      public addPermission(permissionName: string) {
        return this.permissions.add(permissionName);
      }
    }
	
    class Cashier extends Employee {
      public addPermission(permissionName: string) {
        if (permissionName === 'deleteProducts') {;
          throw new Error(
            'Cashier should not be able to delete products!'
          );
        }
        return this.permissions.add(permissionName);
      }
    }
The above example, on the other hand, is fine in terms of typings. Unfortunately, it also breaks the Liskov substitution principle.

    const employee = new Employee();
    employee.addPermission('deleteProducts');
	
    const employee = new Cashier();
    employee.addPermission('deleteProducts');
> Error: Cashier should not be able to delete products!

The same thing applies to output conditions. If the function that we override returns a value, the child class shouldn’t put additional validation on the output.

## Interface segregation principle
The interface segregation principle puts an emphasis on creating smaller and more specific interfaces. Let’s imagine the following situation.

    interface Bird {
      fly(): void;
      walk(): void;
    }
Above, we have an interface of a bird. We assume that birds can walk and fly. It is straightforward to create an example of such a bird:

    class Nightingale implements Bird {
      public fly() {
        /// ...
      }
      public walk() {
        /// ...
      }
    }
The above is not always the case, though. The above assumption can be incorrect.

    class Kiwi implements Bird {
      public fly() {
        throw new Error('Unfortunately, Kiwi can not fly!');
      }
      public walk() {
        /// ...
      }
    }
The interface segregation principle states that no client should be forced to depend on methods it does not use. By putting too many properties in our interfaces, we risk breaking the above rule.

What we might do instead is to implement smaller interfaces, sometimes referred to as role interfaces.

    interface CanWalk {
      walk(): void;
    }
	
    interface CanFly {
      fly(): void;
    }
	
    class Nightingale implements CanFly, CanWalk {
      public fly() {
        /// ...
      }
      public walk() {
        /// ...
      }
    }
	
    class Kiwi implements CanWalk {
      public walk() {
        /// ...
      }
    }
By changing our approach to interfaces, we avoid bloating them and make our software easier to maintain.

## Dependency inversion principle
The core of the dependency inversion principle is that HIGH-LEVEL MODULES SHOULD NOT DEPEND ON THE LOW-LEVEL MODULES. Instead, both of them should depend on abstractions.

Let’s create an example to investigate the above further.

    interface Person {
      introduceSelf(): void;
    }
	
    class Engineer implements Person {
      public introduceSelf() {
        console.log('I am an engineer');
      }
    }
	
    class Musician implements Person {
      public introduceSelf() {
        console.log('I am a musician');
      }
    }
The above behavior is elementary and academic, but it is not always the case. If the introduction was more complicated, we might want to create separate classes just for that.

    interface IntroductionService {
      introduce(): void;
    }
	
    class EngineerIntroductionService implements IntroductionService {
      public introduce() {
        console.log('I am an engineer');
      }
    }
	
    class Engineer implements Person {
      private introductionService = new EngineerIntroductionService();
      public introduceSelf() {
        this.introductionService.introduce();
      }
    }
Unfortunately, the above code breaks the dependency inversion principle. It says that we should invert the dependencies of the  `Engineer` and the  `EngineerIntroductionService`.

    class Engineer implements Person {
      public introductionService: EngineerIntroductionService;
     
      constructor(introductionService: IntroductionService) {
        this.introductionService = introductionService;
      }
     
      public introduceSelf() {
        this.introductionService.introduce();
      }
    }
	
    const engineer = new Engineer(new EngineerIntroductionService());
The benefit of the above is that we don’t need subclasses for the Engineer and the Musician.

    class Person {
      public introductionService: IntroductionService;
     
      constructor(introductionService: IntroductionService) {
        this.introductionService = introductionService;
      }
	  
      public introduceSelf() {
        this.introductionService.introduce();
      }
    }
	
    const engineer = new Person(new EngineerIntroductionService());
    const musician = new Person(new MusicianIntroductionService());
The above approach is an example of using COMPOSITION INSTEAD OF INHERITANCE.

Also, this makes our classes easier to unit-test, because we can effortlessly provide a mocked service in a constructor.

## Summary

By applying SOLID PRINCIPLES to our code, we can make it more readable and maintainable. Also, those are good practices that make our codebase easier to expand while not affecting other parts of our application.

Even though SOLID was defined quite some time ago, it shows that it is still relatable, and we can gain some real benefits from understanding them.

