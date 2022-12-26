# Samsung PRISM 2021 Report

-  Worklet - Automated Test Case Generation and Prediction in Java/Kotlin </b>

~ Report by Praddyumn Shukla <br>
~ First Draft updated on Github:  27th December, 2022

## Problems faced in generating automated test cases:
- How do we generate the input data?
- How to we detect a bug in the code? (because assertions on return value just capture the current behaviour)

## Various approaches followed in increasing order of the complexity:

### 1. Random Testing: 
- It fails to cover the edge case for divide where it is very less possible to generate a value = 0 for y randomly. For example:
```
double divide(int x , int y)
{
  if(y == 0)
    throw new ArithmeticException;
  return x / y;
}
```
- The code coverage is very poor since only 1 out of 4 billiom Test Cases contain a 0.

### 2. Seeding:
+ Here we try to create a pool of exisiting values in the code and pick the input test case value from this pool itself.
+ 50% of the time we pick randomly and 50% of the time we pick from the pool so we can solve the previous divide issue.
+ But it fails on something like:
```
bool solve(int x)
{
  return (x * 2 == 42)
}
```
Because here the answer is 21 but the pool of exisiting values only contain 2 and 42 so we are never able to reach the answer.

### 3. Optimization problem:
Since the goal is to optimize the code coverage hence we convert test generation to an optimization problem and use search algorithms (genetic algo/simulated annealing) to find the best test suites(set of test cases)

There are mainly 3 parameters for this search algorithm:

<b> a) Search space - </b> set of all possible test case values. 

For ex. in the divide code, the space of all possible integer pairs is the search space but it's not necessary that the code is always this trivial. In majority of the cases we develop Object Oriented Software which has a sequence of Function calls as the possible search space.

<b> b) Search operations - </b> modify the test case values to explore the search space. Because when you start with 500 you go to 1000 and do not directly jump to 1 Billion. Modifications like:
- remove a statement
- adding a random statement
- add/remove some value in case of integer
- modify one character in case of string

<b> c) Fitness function - </b> which evaluates how good a test suite/case is. Questions asked by this function:
- Does this cover new instructions? (this is tough to estimate)
- How close it is to cover a new instructions?

For example, if there is a statement like ``` x == 42 ``` if you start with 1 and 100 then you realize that 1 is more closer to 42 as compared to 100 hence you try to explore the space around 1 so that you can satisfy the given constraint faster.

We call this as Heuristic method to find the branch distance(distance from the constraint) </br>
This can be achieved through Byte Code instrumentation by modifying each branch condition. 

This branch distance is a boolean predicate which means that when it becomes 0, a constraint is satisfied. </br>
For strings, there are methods like  ```x.startsWith(y)``` for calculating this distance.


## Understanding ByteCode:

In C language, the compiler takes the a1.c , a2.c files and produces their object files like a1.obj , a2.obj 
- These object files consist of the machine code. 
- The linker then clubs these object files together and produces an .exe file 
- The loader loads this .exe file into the RAM for execution

Whereas in Java, the compiler takes the a1.java , a2.java files and produces the corresponding class files like a1.class , a2.class 
- These object files consist of the byte code. 
- Unlike C, no linking is done. The JVM resides on the RAM and the class loader brings the class files dynamically on the RAM (based on the requirement instead of bringing everyone) 
- The byte code is then verified before giving it to the execution engine of that particular machine.
- The execution enginer then converts it into the native machine code. (Just In Time compiling - JIT) due to which Java is comparatively slow.

Reference: https://youtu.be/G1ubVOl9IBw

##

The Optimization problem minimizes all the branch distances to satisfy the constraints. Some ways to obtain it are:

1. Hill climbing - linearly move towards the target. </br>
For ex. in ```x == 42``` if we start with x = 15, the next value we can check for x is either 14 or 16 but since we know 16 is closer to 42 as compared to 14 hence we chose the value 16 for x. This way we move towards 42.

But this might fail on cases with more than one constraints like : ```(x == 2) && (x * y == 42)``` </br>
Here, a good starting point is x = 7 and y = 6 so that one constraint is solved i.e. 7 * 6 = 42, so we have a partial solution. </br>
Now if we do +-1 on x, it is not going to improve our fitness because we might be getting closer to the constraint x == 2 but then we'll be getting away from the other constraint.

2. Genetic Algorithms:
- It keeps a population of solution and does mutation on them to generate the newer solutions.
- Calculate fitness of each solutions
- Keep the good ones and remove the worsts.

SUT = System Under Test </br>
AST Parser was used to generate the Control Flow Graph and Control Dependency Graph via Byte Code Instrumentation. </br>
Reference: https://youtu.be/sRTnBNErq3A
