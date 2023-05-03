Download Link: https://assignmentchef.com/product/solved-cs2030s-lab-8-logging
<br>
In this lab, we are going to write an immutable Logger class to handle the context of logging changes to values while they are operated upon. In a way, the logger separates the code for logging from the main processing. Each logger wraps around a value, and everytime map or flatMap is called, a new Logger object is created. The printlog method can be invoked the output the log.

A sample logging session is shown below:

jshell&gt; Logger.make(5)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).printlog()Value initialized. Value = 5jshell&gt; Logger.make(5).map(x -&gt; x + 1)$.. ==&gt; Logger[6]jshell&gt; Logger.make(5).map(x -&gt; x + 1).printlog()Value initialized. Value = 5Value changed! New value = 6jshell&gt; Logger.make(5).map(x -&gt; x * 1)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).map(x -&gt; x * 1).printlog()Value initialized. Value = 5Value unchanged. Value = 5

Notice that the log of value changes through time is output whenever the printlog() method is called. There are three different types of log messages:

During initialization: Value initialized. Value = ..When value is modified: Value changed! New value = ..When value remains unchanged: Value unchanged. Value = ..TaskYour task is to write a Logger class that provides the operations make, equals, printlog, map, flatMap and test. You will also write several applications using the Logger as solutions to classic computation problems. This would allow us to look at the values changes when solving each problem.

This task is divided into several levels. Read through all the levels to see how the different levels are related.

Remember to:

always compile your program files first before using jshell to test your programuse checkstyle and javadoc comments to enhance code readability and facilitating code review

Level 1Creating the loggerStart by writing a static method make to wrap a value within a Logger. Include the toString() method as well as the printlog() method to return the string representation of the Logger and output the log messages respectively. At this point of time, there are only initialization messages. Finally, include an equals method that returns true if the argument Logger object is the same as this, or false otherwise. Two Loggers are equal if and only if both the wrapped value as well as the logs are the same.

jshell&gt; Logger.make(5)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).printlog()Value initialized. Value = 5jshell&gt; Logger.make(“hello”)$.. ==&gt; Logger[hello]jshell&gt; Logger.make(“hello”).printlog()Value initialized. Value = hellojshell&gt; Logger.make(5).equals(Logger.make(5))$.. ==&gt; truejshell&gt; Logger.make(5).equals(5)$.. ==&gt; falsejshell&gt; Logger.make(5).equals(Logger.make(“five”))$.. ==&gt; falsejshell&gt; Logger.make(5).equals((Object)(Logger.make(5)))$.. ==&gt; truejshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test1.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 2Creating a valid loggerWe need to prevent the make method from being misused by nesting or chaining the methods, e.g.

Logger.make(Logger.make(5))Logger.make(5).make(7)In the case of nesting make methods, we throw an IllegalArgumentException everytime the make method takes in another Logger as argument.

To prevent chaining, we can make the Logger an interface with the static method make, and let LoggerImpl be the concrete class that implements the functionalities of logging. Here, we restrict the chaining of make method calls at the expense of introducing a cyclic dependency between Logger and LoggerImpl… that is, unless someone comes up with a better idea…

In addition, passing a null to make will also result in an IllegalArgumentException thrown.

jshell&gt; Logger.make(5)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5) instanceof Logger$.. ==&gt; truejshell&gt; try { Logger.make(Logger.make(7)); } catch (Exception e) { System.out.println(e); }java.lang.IllegalArgumentException: already a Loggerjshell&gt; Logger.make(5).make(7)|  Error:|  illegal static interface method call|    the receiver expression should be replaced with the type qualifier ‘Logger&lt;java.lang.Integer&gt;’|  Logger.make(5).make(7)|  ^——————–^jshell&gt; try { Logger.make(null); } catch (Exception e) { System.out.println(e); }java.lang.IllegalArgumentException: argument cannot be nulljshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test2.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 3The map methodInclude a map method that takes in a function of the form Function&lt;? super T, ? extends U&gt;, applies it on the value, and wraps the result in a Logger.

jshell&gt; Logger.make(5)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).printlog()Value initialized. Value = 5jshell&gt; Logger.make(5).map(x -&gt; x + 1)$.. ==&gt; Logger[6]jshell&gt; Logger.make(5).map(x -&gt; x + 1).printlog()Value initialized. Value = 5Value changed! New value = 6jshell&gt; Logger&lt;Integer&gt; a = Logger.make(5)jshell&gt; a.printlog()Value initialized. Value = 5jshell&gt; a.map(x -&gt; x + 1)$.. ==&gt; Logger[6]jshell&gt; a.printlog()Value initialized. Value = 5jshell&gt; Logger.make(5).map(x -&gt; x)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).map(x -&gt; x).printlog()Value initialized. Value = 5Value unchanged. Value = 5jshell&gt; Logger.make(5).equals(Logger.make(5).map(x -&gt; x))$.. ==&gt; falsejshell&gt; Logger.make(5).map(x -&gt; x).equals(Logger.make(5))$.. ==&gt; falsejshell&gt; Logger.make(5).map(x -&gt; x + 1).map(x -&gt; x – 1)$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).map(x -&gt; x + 1).map(x -&gt; x – 1).printlog()Value initialized. Value = 5Value changed! New value = 6Value changed! New value = 5jshell&gt; Logger.make(5).map(x -&gt; x + 1).map(x -&gt; x – 1).equals(Logger.make(5))$.. ==&gt; falsejshell&gt; Logger.make(“hello”).map(String::length)$.. ==&gt; Logger[5]jshell&gt; Logger.make(“hello”).map(String::length).printlog()Value initialized. Value = helloValue changed! New value = 5jshell&gt; Function&lt;Object, Boolean&gt; f = x -&gt; x.equals(x)jshell&gt; Logger.make(“hello”).map(f)$.. ==&gt; Logger[true]jshell&gt; Function&lt;String, Number&gt; g = x -&gt; x.length();jshell&gt; Logger&lt;Number&gt; lognum = Logger.make(“hello”).map(g)jshell&gt; lognumlognum ==&gt; Logger[5]jshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test3.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 4The flatMap methodWe are now ready to write the flatMap method that takes a function of the form Function&lt;? super T, ? extends Logger&lt;? extends U&gt;&gt;. Pay special attention to how the logs are to be combined.

jshell&gt; Logger.make(5).printlog()Value initialized. Value = 5jshell&gt; Logger.make(5).flatMap(x -&gt; Logger.make(x))$.. ==&gt; Logger[5]jshell&gt; Logger.make(5).flatMap(x -&gt; Logger.make(x)).printlog()Value initialized. Value = 5jshell&gt; Logger.make(5).flatMap(x -&gt; Logger.make(x)).equals(Logger.make(5))$.. ==&gt; truejshell&gt; Logger&lt;Integer&gt; a = Logger.make(5).flatMap(x -&gt; Logger.make(x).map(y -&gt; y + 2)).flatMap(y -&gt; Logger.make(y).map(z -&gt; z * 10))jshell&gt; Logger&lt;Integer&gt; b = Logger.make(5).flatMap(x -&gt; Logger.make(x).map(y -&gt; y + 2).flatMap(y -&gt; Logger.make(y).map(z -&gt; z * 10)))jshell&gt; a.printlog()Value initialized. Value = 5Value changed! New value = 7Value changed! New value = 70jshell&gt; b.printlog()Value initialized. Value = 5Value changed! New value = 7Value changed! New value = 70jshell&gt; a.equals(b)$.. ==&gt; truejshell&gt; Logger&lt;Integer&gt; c = Logger.make(5).map(x -&gt; x + 2).map(x -&gt; x * 10)jshell&gt; a.equals(c)$.. ==&gt; truejshell&gt; Function&lt;Object, Logger&lt;Boolean&gt;&gt; f = x -&gt; Logger.make(x).map(y -&gt; y.equals(y))jshell&gt; Logger.make(“hello”).flatMap(f)$.. ==&gt; Logger[true]jshell&gt; Function&lt;String, Logger&lt;Number&gt;&gt; g = x -&gt; Logger.make(x).map(y -&gt; y.length())jshell&gt; Logger&lt;Number&gt; lognum = Logger.make(“hello”).flatMap(g)jshell&gt; lognumlognum ==&gt; Logger[5]jshell&gt; /exit

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test4.jsh

Check your styling by issuing the following

$ checkstyle *.java

Level 5Let’s write some applications using JShell that makes use of our Logger so as to observe how the values changes over the course computation. Save your methods in the file level5.jsh.

Define an add(Logger&lt;Integer&gt; a, int b) method that returns the result of a added to b wrapped in a Logger that preserves the log of all operations of a, as well as the addition to b.

jshell&gt; add(Logger.make(5), 6)$.. ==&gt; Logger[11]jshell&gt; add(Logger.make(5), 6).printlog()Value initialized. Value = 5Value changed! New value = 11jshell&gt; add(Logger.make(5).map(x -&gt; x * 2), 6)$.. ==&gt; Logger[16]jshell&gt; add(Logger.make(5).map(x -&gt; x * 2), 6).printlog()Value initialized. Value = 5Value changed! New value = 10Value changed! New value = 16

The sum of non-negative integers from 0 to n (inclusive of both) can be computed recursively using

int sum(int n) {if (n == 0) {return 0;} else {return sum(n – 1) + n;}}

Redefine the above method such that it returns the result wrapped in a Logger. You may find the above add method useful.jshell&gt; sum(0)$.. ==&gt; Logger[0]jshell&gt; sum(0).printlog()Value initialized. Value = 0jshell&gt; sum(5)$.. ==&gt; Logger[15]jshell&gt; sum(5).printlog()Value initialized. Value = 0Value changed! New value = 1Value changed! New value = 3Value changed! New value = 6Value changed! New value = 10Value changed! New value = 15

The Collatz conjecture (or the 3n+1 Conjecture) is a process of generating a sequence of numbers starting with a positive integer that seems to always end with 1.

int f(int n) {if (n == 1) {return 1;} else if (n % 2 == 0) {return f(n / 2);} else {return f(3 * n + 1);}}

Write the function f that takes in n and returns a Logger&lt;Integer&gt; that wraps around the final value, with a log of the value changes over time. You should include a test method in the Logger that takes in a Predicate and returns true if the value within the Logger satisfies the predicate, and false otherwise.

jshell&gt; Logger.make(5).test(x -&gt; x == 5)$.. ==&gt; truejshell&gt; Logger.make(5).map(x -&gt; x + 1).test(x -&gt; x % 2 != 0)$.. ==&gt; falsejshell&gt; Logger.make(“hello”).test(x -&gt; x.length() == 5)$.. ==&gt; truejshell&gt; f(16)$.. ==&gt; Logger[1]jshell&gt; f(16).printlog()Value initialized. Value = 16Value changed! New value = 8Value changed! New value = 4Value changed! New value = 2Value changed! New value = 1jshell&gt; f(10)$.. ==&gt; Logger[1]jshell&gt; f(10).printlog()Value initialized. Value = 10Value changed! New value = 5Value changed! New value = 15Value changed! New value = 16Value changed! New value = 8Value changed! New value = 4Value changed! New value = 2Value changed! New value = 1

Check the format correctness of the output by running the following on the command line:

$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test5.jsh

Check your styling by issuing the following

$ checkstyle *.java