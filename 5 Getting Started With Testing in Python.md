# Getting Started With Testing in Python

By: Anthony Shaw  

Last Updated: early 2019  

Source: https://realpython.com/python-testing/  

### Intro

This tutorial is for anyone who has written a fantastic application int Python but ahs not yet written any tests.  

Testing in Python is a huge topic and can come with a lot of complexity, but it does not need to be hard. You can get started creating simple tests for your application in a few easy steps and then build on it from there.  

In this tutorial, you will learn how to create a basic test, execute it, and find the bugs before your users do! You will learn about the tools available to write and executre tests, check your application's performance, and even look for secuirty issues.  

### Testing your code

There are many ways to test your code. In this tutorial, you will leran the techniques from the most basic steps and work towards advanced methods.  

##### Automated vs. Manual Testing

The good news is, you have probably already created a test without realizing it. Exploratory testing is a form of manual testing. Exploratory testing is a form of testing that is done without a plan. In an exploratory testm you are just exploring the applications.  

To have a complete set of manual tests, all you need to do is make a list of all the features your application has, the different types of inpt it can accept, and the expected results. Now every time you make a change to your ode, you need to go through every single item on that list and check it.  

This is where automated testing comes in. Automated testing is the execution of your test plan (the parts of your application you want to test, the order in which you want to test them, and the expected responses) by a script instead of a human. Python already comes with a s et of tools and libraries to help you create automated tests for your application. We will explore those tools and libraries in this tutorial.  

### Unit test vs integration tests

The world of testing has no shortage of terminology, and now that you know the difference betwee automated and manual testing, it is time to go a level deeper.  

Think of how you might test the lights on a car. You would turn on the lights (known as the test step) and go outside the car or ask a friend to check that the lights are on (known as the test assertion). Tessting multiple components is known as integration testing.  

Think of all the things that need to work correctly in order for a simple task to give the right result. These components are like the parts to  your application, all of those classes, functions, and modules you have written.  

A major challenge with inegration testing is when an integration test does not give the right result. It is very hard to diagnose the issue without being able to isolate which part of the ststem is failing. If the lights did not turn on, then maybe the bulbs are broken. Is the battery dead? What about the alternator? Is the car's computer failing?  

If you have a fancy modern car, it will tell you when your light bulbs have gone. It does this using a form of unit test.  

A unit test is a smaller test, one that checks that a single component operates in the right way. A unit test helps you to isolate what is broken in your application and fix it faster.  
You have just seen two types of tests:  
1. An integration test checks that components in your application operate with each other
2. A unit test checks a small component in your application.  

You can write both integration and unit tests in Python. To write a unit test for the built-in function sum(), you would check the output of sum() against a known output.  

For example, here's how you check that sum() of the numbers 1,2,3 equal 6

<p>
    assert sum([1, 2, 3]) == 6, "Should be 6"
</p>

This will not output anything because the values are correct.  

If the result from sum() is incorrect, this will fail with an AssertionError and the message "Should be 6:". Try an assertion statement again with the wrong values to see an AssertionError:  
<p>

    assert(sum([1, 1, 1,]) == 6), 'should be 6'
    Traceback (most recent call last):
        File "stdin", line 1 in module
    AssertionError: Should be 6
  
    In the REPL, you are seeing the raised AssertionError because sum does not match 6. Instead of testing on the REPL, you want to put this into a new Python file called test_sum.py and execute it again:  

    def test_sum():
        assert sum([1, 2, 3]) == 6, "Should be 6"

    if __name__ == "__main__":
        test_sum()
        print("Everything passed")
    
    Now you have written a test case, an assertion, and an entry point (the command line). You can now execute this at the command line:  
    
    python test_sum.py
    Everything Passed
    
    In Python, sum() accepts any iterable as its first argument. You tested with a list. Now test with a tuple as well. You can write another test file test_sum_2.py.  
    
    def test_sum():
    assert sum([1, 2, 3]) == 6, "Should be 6"

    def test_sum_tuple():
        assert sum((1, 2, 2)) == 6, "Should be 6"

    if __name__ == "__main__":
        test_sum()
        test_sum_tuple()
        print("Everything passed")
    
    When you execute test_sum_2.py, the script will give an error because the sum() of (1,2,2) is 5, not 6. The result of the script gives you the error message, the line of code, and the traceback:  
    
    python test_sum_2.py
    Traceback (most recent call last):
      File "test_sum_2.py", line 9, in module
        test_sum_tuple()
      File "test_sum_2.py", line 5, in test_sum_tuple
        assert sum((1, 2, 2)) == 6, "Should be 6"
    AssertionError: Should be 6
    
    Here you can see how a mistake in your code gives an error on the console with some information on where was and what the expected result was.  
    
    Writing tests in this way is okay for a simple check, but what if more that one fails? This is where test runners come in. The test runner is a special application designed for running tests, checking the output, and giving you tools for debuggin and diagnosing tests and applications.  
    
</p>  

### Choosing a test runner

There are many test runners available for Python. The one built into the Python standard library is called unittest. In this tutorial, you will be using unittest test cases and the unittest test runner. The priciples of unittest are easily protable to other frameworks. The three most popular test runners are:  
- unittest
- nose or nose2
- pytest

Choosing the best runner for your requirements and level of experience is important.  

### Unittest

Unittest has been built into the Python standard library since version 2.1. You will probably see it in commercial Python applications and open-source projects.  

Unittest contains both a testing framework and a test runner.unittest has some important requirements for writing and executing tests.  

unittests requires thats:
- you put your tests into classes as methods
- you use a series of spceial assertion methods in the unittest.TestCase class instead of built-in assert statement

To convert the earlier example to a unittest test case, you would have to:
1. import unittest from the standard library
2. create a class called TestSum that inherits from the TestCase class
3. convert the test functions into methods by adding self as the first argument
4. change the assertion to use the self.assertEqual() method on the TestCase class
5. Change the command-line entry point to call unittest.main()

Follow those steps by creating a new file test_sum_unittest.py with the following code: 

<p>
    import unittest


    class TestSum(unittest.TestCase):

        def test_sum(self):
            self.assertEqual(sum([1, 2, 3]), 6, "Should be 6")

        def test_sum_tuple(self):
            self.assertEqual(sum((1, 2, 2)), 6, "Should be 6")

    if __name__ == '__main__':
        unittest.main()
    
    if you execute this at the command line, you will see one success (indicated with.) and one failure (indicated with F):  
    
    python test_sum_unittest.py
    .F
    ======================================================================
    FAIL: test_sum_tuple (__main__.TestSum)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "test_sum_unittest.py", line 9, in test_sum_tuple
        self.assertEqual(sum((1, 2, 2)), 6, "Should be 6")
    AssertionError: Should be 6

    ----------------------------------------------------------------------
    Ran 2 tests in 0.001s

    FAILED (failures=1)
    
</p>

### Writing your first test
<p>
    Lets bring together what you have learned so far and, instead of testing the buil-in sum() function, test a simple implementation of the same requirement.  

    Create a new project folder and, inside that, create a new folder called my_sum. Inside my_sum, create an empty file called __init__.py. Create the __init__.py file means that my_sum folder can be imported as a module from the parent directory.  

    Your project should look like this  

    project/
    │
    └── my_sum/
        └── __init__.py

    open up my_sum/__init__.py and create a new function called sum(), which takes an iterable (a list, tuple, or set) and adds the values together:  
    
    def sum(arg):
    total = 0
    for val in arg:
        total += val
    return total
    
    This code example creates a variable called total, iterates over all the values in arg, and adds them to total. It then returns the result once the iterable has been exhausted.  
    
</p>

### Where to write the test

<p>
    To get started writing tests, you can simply create a file called test.py, which will contain your first test case. Because the file will need to be able to import your applications to be able to test it, you want to place test.py above the package folder, so your directroy tree will look something like this:  

    project/
    │
    ├── my_sum/
    │   └── __init__.py
    |
    └── test.py
    
    You will find that, as you add more and more tests, your single file will become cluttered and hard to maintain, so you can create a folder called tests/ and split the tests into multiple files. It is convention to ensure each file starts with test_ so all test runners will assume that Python file contains tests to be executed. Some very large projects split tests into more subdirectories based on their purpose or usage.  
    
</p>

### How to structure a simple test

Before you dive into writing tests, you wil want to first make a couple decisions:
1. What do you want to test?
2. Are you wruting a unit test or an integration test?

Then the sturcute of a test should loosely follow this workflow:  
1. create your inputs
2. execute the code being tested, captureing the output
3. compare the output with an expected result

For this application, you are testing sum(). There are many behaviors in sum() you could check such as:  
- can it sum a list of whole numbers (integers)>
- can it sum a tuple of set?
- can it sum a list of floats?
- what happens when you provide it with a bad value, such as a single integer or a string?
- what happens when one of the values is negative?

The most simple test would be a list of integers. Create a file, test.py with the following Python code:  

<p>
    import unittest

    from my_sum import sum


    class TestSum(unittest.TestCase):
        def test_list_int(self):
            """
            Test that it can sum a list of integers
            """
            data = [1, 2, 3]
            result = sum(data)
            self.assertEqual(result, 6)

    if __name__ == '__main__':
        unittest.main()
    
</p>

This code example:  
1. imports sum() from the my_sum package you created
2. defines a new test case class called TestSum, which inherits from unittest.TestCase 
3. defines a test method, .test_list_int(), to a list of integers. the method .test_list_int() will:  
- declare a variable data with a list of numbers (1, 2, 3)
- assign the results of my_sum.sum(data) to a result variable
- assert that the value of result equals 6 by using the .assertEqual() method on the unittest.TestCase class
4. defines a command-line entry point, which runs the unittest test-runner.main()  


</p>

### How to write assertions

The last step of writing a test is to validate the output against a known response. this is known as assertion. THere are some general best practices around how to write assetions:  
- make sure tests are repeateable and run your test multiple times to make sure it gives the same result every time
- try and assert results that relate to your input data, such as checking that the result is the actualy sum of values in the sum() example

unittest comes wiht lots of methods to assert on the values, types, and existence of variables. Here are some of the most commonly used methods:  
<p>
    
<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-cly1{text-align:left;vertical-align:middle}
.tg .tg-0lax{text-align:left;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-cly1">Method</th>
    <th class="tg-cly1">Equivalent to</th>
  </tr>
  <tr>
    <td class="tg-cly1">.assertEqual(a, b)</td>
    <td class="tg-cly1">a == b</td>
  </tr>
  <tr>
    <td class="tg-cly1">.assertTrue(x)</td>
    <td class="tg-cly1">bool(x) is True</td>
  </tr>
  <tr>
    <td class="tg-cly1">.assertFalse(x)</td>
    <td class="tg-cly1">bool(x) is False</td>
  </tr>
  <tr>
    <td class="tg-cly1">.assertIs(a, b)</td>
    <td class="tg-cly1">a is b</td>
  </tr>
  <tr>
    <td class="tg-cly1">.assertIsNone(x)</td>
    <td class="tg-cly1">x is None</td>
  </tr>
  <tr>
    <td class="tg-0lax">.assertIn(a, b)</td>
    <td class="tg-0lax">a in b</td>
  </tr>
  <tr>
    <td class="tg-0lax">.assetIsInstance(a, b)</td>
    <td class="tg-0lax">isinstance(a, b)</td>
  </tr>
</table>

there are also oppisite methods like .assertIsNot(), by including Not as the end.
</p>

### Side effects

When you are writing tests, it is often not as simple as looking at the return value of a function. Often, executing a piece of code will alter other things in the enviornment, such as the attribute of a class, a file on the filesystem, or a value in a database. Tese are known as side effects and are important part of testing. Decide if the side effect is being tested before including it in your list of assertions.  

If you find that the unit code you want to test has lots of side effects, you might be breaking the single responsibility principle. Breaking the single responsibilty principle means the piece of code is doing too many things and would be better of being refactored. Following the single responsibility priniciple is a greay way to design code that is easy to write repeateable and simple unit tests for, and ultimately, reliable applications.  

### Executing your first test

Now that you have created the first test, you want to execute it. Sure you know it is going to pass, but before you create more complex tests, you should check that you can execute the tests successfully.  

### Executing test runners

<p>
    The Python application that executes your test code, checks the assertions , and give you test results in your console is called the test runner.  

    At the bottom of test.py, you added this small snippet of code:
    
    if __name__ == '__main__':
        unittest.main()
    
    This is a command line entry point. It means you execute the scirpt alone by running python test.py at the command line, it will call unittest.main(). This executes the test runner by discovering alll classes in this file that inherit from unittest.TestCase.  
    
    This is one of many ways to execute the unittest test runner. WHen you have a singe test file named test.py, callling python test,oy is a greay way to get started.  
    
    Another way is using the unittest command line. Try this:  
    
    python -m unittest test
    
    This will execute the same test module (called test) via the command line. 
    
    You can provide additional options to change the output. One of those is -v for verbose. Try that next:  
    
    python -m unittest -v test
    test_list_int (test.TestSum) ... ok

    ----------------------------------------------------------------------
    Ran 1 tests in 0.000s
    
    
   This executed the one test inside test.py and printed the results to the console. Verbose mode listed the names of the tests it executed first, along with the results of each test.  
    
    Instead of providing the name of a module containing tests, you can request an auto-discovery using the following:
    
    python -m unittest discover
    
    This will search the current directory for any files names tests*.py and attempt to test them. 
    
    Once you have multiple test files, as long as you follow the test*.py naming pattern, you can provide the name of the directorying instead by using the -s flag and the name of the directory: 
    
    python -m unittest discover -s tests
    
    unittest will run all tests in a single tes plan and give you the results.
    
    Lastly, if your source code is not in the directory root and ocntained in a subdirectory, for example, in a folder called src/, you can tell unittest where to execute the tests so that it can import the modules correctly with the -t flag:
    
    python -m unittest discover -s tests -t src
    
    unittest will change to the src/ directory, scan for all test*.py files inside the tests directory, and execute them.  
    
</p>

### Understanding test output

<p>
    That was a very simple example where everything passes, so now you are going to try a failing test and interpret the output.  

    sum() should be able to accept other lists of numeric types, like fractions. 

    At the top of the test.py file, add an important statement to import the Fraction type from the fraction module in the standard library:  

    from fractions import Fraction
    
    Now add a test with an assertion expecting the incorrect values, in this case expecting the sum of 1/4, 1/4 and 2/5 to be 1:
    
    import unittest
    
    from my_sum import sum
    
    class TestSum(unittest.TestCase):
        def test_list_int(self):
            '''
                Test that it can sum a list of integers
            '''
        data = [1, 2, 3]
        result = sum(data)
        self.assertEqual(result, 6)
    
        def test_list_fraction(self):
            '''
                Test that it can sunm a list of fractions
            '''
            data = [Fraction(1, 4), Fraction(1, 4), Fraction(2, 5)]
            result = sum(data)
            self.assertEqual(result, 1)
    
    if __name__ == '__main__':
        unittest.main()
    
    
    If you execute the test again with python -m unittest test, you should see the following output:
    
    python -m unittest test
    F.
    ======================================================================
    FAIL: test_list_fraction (test.TestSum)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "test.py", line 21, in test_list_fraction
        self.assertEqual(result, 1)
    AssertionError: Fraction(9, 10) != 1

    ----------------------------------------------------------------------
    Ran 2 tests in 0.001s

    FAILED (failures=1)

</p>
In the output, you will see the following information:

1. The first line shows the execution results of all the tests, one failed (F) and one passed(.).
2. The FAIL entry shows some details about the failed test:
    - the test method name (test_list_fractions)
    - the test module (test) and the test case (testSum)
    - a traceback to the failing line
    - the details o the assertion with the expected results (10 and the actual result (Faction(9, 10)))
    
Remember, you can add extra information to the test output by adding the -v flag to the python -m unittest command. 
    
### Running your tests from PyCharm

If you are using the PyCharm IDE, you can run unittest or pytest by following these steps:
1. In the project window, select the tests directory
2. on the context menu, choose the run command for unittest. For example, choose run unittest in my tests

This will execute unittest in a test window and give you the results within PyCharm


### More advanced testing scenarios 

Before you step into creating tests for your application, remember the three basics steps of every test:
1. create your inputs
2. execute the code, captureing the output
3. compare the output with an expected result

It is not always as easy as creating a static value for the inpit like a string or a number. Sometimes your application will require an instance or a class or a context. What do you do then?  

The data that you create as input is known as a fixture. It is common practice to crete fixtures and reuse them.  

If you are running the same test and passing different values each time and expecting the same result, this is known as paramaterization.  

### Handling expected failures

<p>
    
    Earlier, when you made a list of scenarios to test sum(), a question came up: what happens when you provide is with a bad value, such as a single integer or a string?  

    In this case, you would expect sum() to throw an error. When it does throw an error, that would cause the test to fail.  

    There is a special way to handle expected erros. You can use .assertRaises() as a context-manager, then inside the block execute the test steps: 
    
    import unittest
    
    from my_sum import sum
    
   class TestSum(unittest.TestCase):
    def test_list_int(self):
        """
        Test that it can sum a list of integers
        """
        data = [1, 2, 3]
        result = sum(data)
        self.assertEqual(result, 6)

    def test_list_fraction(self):
        """
        Test that it can sum a list of fractions
        """
        data = [Fraction(1, 4), Fraction(1, 4), Fraction(2, 5)]
        result = sum(data)
        self.assertEqual(result, 1)

    def test_bad_type(self):
        data = "banana"
        with self.assertRaises(TypeError):
            result = sum(data)

    if __name__ == '__main__':
        unittest.main()
    
    This test case will now only pass if sum(data) raises a TypeError. You can replace TypeError with any exception type you choose.
    
</p>

### Isolating behaviors in your application

![side effects](https://files.realpython.com/media/YXhT6fA.d277d5317026.gif)  

Earlier in the tutorial, you learned what a side effect is. Isde effects make unit testing harder since, each time a test is run, it might give a different result, or even worse, one test could impact the state of the application and cause another test to fail!  

There are some simple techniques you can use to test parts of your application that have many side effects:  
- Refactoring code to follow the Single Responsibility Principle
- Mockking out any method or function calls to remove side effects
- Using integration testing instead of unit testing for this piece of the application

### Writing integration tests

So far, you have been learning mainly about unit testing. Unit testing is a greay way to build predicatble and stable code. But at the end of the day, you application needs to work when it starts!  

Integration testing is the testing of multiple components of the application to check that they working together. Integration testing might require acting like a consumer or user of the application by:  
- calling an HTTP REST API
- calling a Python API
- calling a web service
- running a comand line 

Each of these types of integration tests can be written in the same way as a unit test, following the input, execute, and assert pattern. The most significant difference is that integration tests are checking more components at once and therefore will have more side effects than a unit test. Also, integration tests will require more fixtures to be in place, like a database, a network socket, or a configuration file.  

This is why it is good practice to seperate your unit tests and your integration tests. The creation of fixtures required for an integration like a test database and the test cases themselves often take a longer to execute than unit tests, so you may only want to run integration tests before you push to production instead of once on every commit.  

A simple way to separate unit and integration tests is simply to put them in different folders:  

<p>
    project/
    │
    ├── my_app/
    │   └── __init__.py
    │
    └── tests/
        |
        ├── unit/
        |   ├── __init__.py
        |   └── test_sum.py
        |
        └── integration/
            ├── __init__.py
            └── test_integration.py

    There are many ways to execute only a selected group of tests. The specify source directory flag, -s, can be added to unittest discover with the pth containing the tests:
    
    python -m unittest discover -s tests/integration
    
    unit test will ahve given you the result of all the tests within the test/integration directory. 
    
</p>

### Testing Data-Driven Applications!!!

Many integration tests will require backend data like database to exist with certain values. For example, you might want to ahve a test that checks that the application displayes correctly with more than 100 customers in the database, or the order page works even if the product names are displayed in Japanese.  

These types of integration tests will depend on different test fixures to make sure they are repeatable and predictable.  

A good technique to use is to store the test data in a folder within your integration testing folder called fixtures to indicate that it contains test data. Then, within your tests, you can load the data and run the test.  

Here is an example of that strucute if the data consisted of JSON files:

<p> 
    project/
    │
    ├── my_app/
    │   └── __init__.py
    │
    └── tests/
        |
        └── unit/
        |   ├── __init__.py
        |   └── test_sum.py
        |
        └── integration/
            |
            ├── fixtures/
            |   ├── test_basic.json
            |   └── test_complex.json
            |
            ├── __init__.py
            └── test_integration.py
    
    Within your test case, you can use the .setUp() method to load the test data from a fixture file in a known path and execute many tests aagainst that test data. Remember you can have multiple test cases in a single Python file, and the unittest discovery will execute both. You can have one test case for each set of test data:  
    
    