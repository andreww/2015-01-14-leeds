---
layout: lesson
root: ../..
---

# Getting the Right Answer

Our previous lessons have introduced the basic tools of programming:
variables and lists,
file I/O,
loops,
conditionals,
and most importantly,
functions.
What they haven't done is show us how to tell if a program is getting the right answer.
For the sake of argument, if each line we write has a 99% chance of being right,
then a 70-line program will be wrong more than half the time.
We need to do better than that,
which means we need to:

* write programs that check their own operation; and
* write tests to catch the mistakes those self-checks miss.

Along the way,
we will learn:

* how Python reports and handles errors;
* how to use a unit testing framework;
* when it's useful to write tests *before* writing code.

## Defensive Programming

We made several mistakes while writing the programs in our first few lessons.
How can we be sure that there aren't still errors lurking in the code we have?
And how can we guard against introducing new errors in code as we modify it?

The first step is to use [defensive programming](../../gloss.html#defensive_programming),
i.e.,
to assume that mistakes *will* happen
and to guard against them.
One way to do this is to add [assertions](../../gloss.html#assertion) to our code
so that it checks itself as it runs.
An assertion is simply a statement that something must be true at a certain point in a program.
When Python sees one,
it checks that the assertion's condition.
If it's true,
Python does nothing,
but if it's false,
Python halts the program immediately
and prints the error message provided.
For example,
this piece of code halts as soon as the loop encounters a value that isn't positive:


	numbers = [1.5, 2.3, 0.7, -0.001, 4.4]
	total = 0.0
	for n in numbers:
    	assert n >= 0.0, 'Data should only contain positive values'
	    total += n
	print 'total is:', total


Programs like the Firefox browser are littered with assertions:
10-20% of the code they contain
are there to check that the other 80-90% are working correctly.
Broadly speaking,
assertions fall into three categories:

-   A [precondition](../../gloss.html#precondition) is something that must be true
    at the start of a function in order for it to work correctly.
-   A [postcondition](../../gloss.html#postcondition) is something that
    the function guarantees is true when it finishes.
-   An [invariant](../../gloss.html#invariant) is something that is always true
    at a particular point inside a piece of code.

For example,
suppose we are representing rectangles using a tuple of four coordinates `(x0, y0, x1, y1)`.
In order to do some calculations,
we need to normalize the rectangle so that it is at the origin
and 1.0 units long on its longest axis.
This function does that,
but checks that its input is correctly formatted and that its result makes sense:


	def normalize_rectangle(rect):
	    #What precondition could we set up here? Hint: how many coordinates do we need?
    	# assert...
    
	    x0, y0, x1, y1 = rect
	    #Another predondition assertion (or maybe two?) - how do we know that X and Y coordinates are correct?
    	# assert...

	    dx = x1 - x0
	    dy = y1 - y0
	    if dx > dy:
    	    scaled = float(dx) / dy
    	    upper_x, upper_y = 1.0, scaled
	    else:
    	    scaled = float(dx) / dy
    	    upper_x, upper_y = scaled, 1.0

	    #Postconditions    
	    assert 0 < upper_x <= 1.0, 'Calculated upper X coordinate invalid'
	    #We may need another one here
    	# assert....

	    return (0, 0, upper_x, upper_y)


The preconditions we set should catch invalid inputs:


	print normalize_rectangle( (0.0, 1.0, 2.0) ) # missing the fourth coordinate
	print normalize_rectangle( (4.0, 2.0, 1.0, 5.0) ) # X axis inverted


The post-conditions help us catch bugs by telling us when our calculations cannot have been correct.
For example,
if we normalize a rectangle that is taller than it is wide everything seems OK:


	print normalize_rectangle( (0.0, 0.0, 1.0, 5.0) )


but if we normalize one that's wider than it is tall,
the assertion is triggered:


	print normalize_rectangle( (0.0, 0.0, 5.0, 1.0) )


Re-reading our function,
we realize that line 10 should divide `dy` by `dx` rather than `dx` by `dy`.
(You can display line numbers by typing Ctrl-M, then L.)
If we had left out the assertion at the end of the function,
we would have created and returned something that looked like a valid answer,
but wasn't;
detecting and debugging that would almost certainly have taken more time in the long run
than writing the assertion.

But assertions aren't just about catching errors:
they also help people understand programs.
Each assertion gives the person reading the program
a chance to check (consciously or otherwise)
that their understanding matches what the code is doing.

Most good programmers follow two rules when adding assertions to their code.
The first is, "[fail early, fail often](../../rules.html#fail-early-fail-often)".
The greater the distance between when and where an error occurs and when it's noticed,
the harder the error will be to debug,
so good code catches mistakes as early as possible.

The second rule is, "[turns bugs into assertions or tests](../../rules.html#turn-bugs-into-assertions-or-tests)".
If you made a mistake in a piece of code,
the odds are good that you have made other mistakes nearby,
or will make the same mistake (or a related one)
the next time you change it.
Writing assertions to check that you haven't [regressed](../../gloss.html#regression)
(i.e., haven't re-introduced an old problem)
can save a lot of time in the long run,
and helps to warn people who are reading the code
(including your future self)
that this bit is tricky.

### Challenges

1.  Suppose you are writing a function called `average` that calculates the average of the numbers in a list.
    What pre-conditions and post-conditions would you write for it?
    Compare your answer to your neighbor's:
    can you think of a function that will past your tests but not hers or vice versa?

2.  Explain in words what the assertions in this code check,
    and for each one,
    give an example of input that will make that assertion fail.
    
    
    	def running(values):
    		assert len(values) > 0
    		result = [values[0]]
    		for v in values[1:]:
        		assert result[-1] >= 0
        		result.append(result[-1] + v)
    		assert result[-1] >= result[0]
    		return result
    

## Exceptions

Assertions help us catch errors in our code,
but things can go wrong for other reasons,
like missing or badly-formatted files.
Most modern programming languages allow programmers to use [exceptions](../../gloss.html#exception) to separate
what the program should do if everything goes right
from what it should do if something goes wrong.
Doing this makes both cases easier to read and understand.

For example,
here's a small piece of code that tries to read parameters and a grid from two separate files,
and reports an error if either goes wrong:


	try:
    	params = read_params(param_file)
    	grid = read_grid(grid_file)
	except:
    	log.error('Failed to read input file(s)')
    	sys.exit(ERROR)


We join the normal case and the error-handling code using the keywords `try` and `except`.
These work together like `if` and `else`:
the statements under the `try` are what should happen if everything works,
while the statements under `except` are what the program should do if something goes wrong.

We have actually seen exceptions before without knowing it,
since by default,
when an exception occurs,
Python prints it out and halts our program.
For example,
trying to open a nonexistent file triggers a type of exception called an `IOError`,
while an out-of-bounds index to a list triggers an `IndexError`:


	open('nonexistent-file.txt', 'r')
	values = [0, 1, 2]
	print values[999]


We can use `try` and `except` to deal with these errors ourselves
if we don't want the program simply to fall over:


	try:
    	reader = open('nonexistent-file.txt', 'r')
	except IOError:
    	print 'Whoops!'


When Python executes this code,
it runs the statement inside the `try`.
If that works, it skips over the `except` block without running it.
If an exception occurs inside the `try` block,
though,
Python compares the type of the exception to the type specified by the `except`.
If they match, it executes the code in the `except` block.

`IOError` is the particular kind of exception Python raises
when there is a problem related to input and output,
such as files not existing
or the program not having the permissions it needs to read them.
We can put as many lines of code in a `try` block as we want,
just as we can put many statements under an `if`.
We can also handle several different kinds of errors afterward.
For example,
here's some code to calculate the entropy at each point in a grid:


	try:
		params = read_params(param_file)
		grid = read_grid(grid_file)
		entropy = lee_entropy(params, grid)
		write_entropy(entropy_file, entropy)
	except IOError:
		report_error_and_exit('IO error')
	except ArithmeticError:
		report_error_and_exit('Arithmetic error')


Python tries to run the four functions inside the `try` as normal.
If an error occurs in any of them,
Python immediately jumps down
and tries to find an `except` of the corresponding type:
if the exception is an `IOError`,
Python jumps into the first error handler,
while if it's an `ArithmeticError`,
Python jumps into the second handler instead.
It will only execute one of these,
just as it will only execute one branch
of a series of `if`/`elif`/`else` statements.

This layout has made the code easier to read,
but we've lost something important:
the message printed out by the `IOError` branch doesn't tell us
which file caused the problem.
We can do better if we capture and hang on to the object that Python creates
to record information about the error:


	try:
		params = read_params(param_file)
		grid = read_grid(grid_file)
		entropy = lee_entropy(params, grid)
		write_entropy(entropy_file, entropy)
	except IOError as err:
		report_error_and_exit('Cannot read/write' + err.filename)
	except ArithmeticError as err:
		report_error_and_exit(err.message)


If something goes wrong in the `try`,
Python creates an exception object,
fills it with information,
and assigns it to the variable `err`.
(There's nothing special about this variable name&mdash;we can use anything we want.)
Exactly what information is recorded depends on what kind of error occurred;
Python's documentation describes the properties of each type of error in detail,
but we can always just print the exception object.
In the case of an I/O error,
we print out the name of the file that caused the problem.
And in the case of an arithmetic error,
printing out the message embedded in the exception object is what Python would have done anyway.

So much for how exceptions work:
how should they be used?
Some programmers use `try` and `except` to give their programs default behaviors.
For example,
if this code can't read the grid file that the user has asked for,
it creates a default grid instead:


	try:
    	grid = read_grid(grid_file)
	except IOError:
    	grid = default_grid()


Other programmers would explicitly test for the grid file,
and use `if` and `else` for control flow:


	if file_exists(grid_file):
    	grid = read_grid(grid_file)
	else:
    	grid = default_grid()


It's mostly a matter of taste,
but we prefer the second style.
As a rule,
exceptions should only be used to handle exceptional cases.
If the program knows how to fall back to a default grid,
that's not an unexpected event.
Using `if` and `else`
instead of `try` and `except`
sends different signals to anyone reading our code,
even if they do the same thing.

Novices often ask another question about exception handling style as well,
but before we address it,
there's something in our example that you might not have noticed.
Exceptions can actually be thrown a long way:
they don't have to be handled immediately.
Take another look at this code:

	
	try:
    	params = read_params(param_file)
	    grid = read_grid(grid_file)
	    entropy = lee_entropy(params, grid)
	    write_entropy(entropy_file, entropy)
	except IOError as err:
    	report_error_and_exit('Cannot read/write' + err.filename)
	except ArithmeticError as err:
    	report_error_and_exit(err.message)


The four lines in the `try` block are all function calls.
They might catch and handle exceptions themselves,
but if an exception occurs in one of them that *isn't* handled internally,
Python looks in the calling code for a matching `except`.
If it doesn't find one there,
it looks in that function's caller,
and so on.
If we get all the way back to the main program without finding an exception handler,
Python's default behavior is to print an error message like the ones we've been seeing all along.

This rule is the origin of the rule "Throw Low, Catch High."
There are many places in our program where an error might occur.
There are only a few, though, where errors can sensibly be handled.
For example,
a linear algebra library doesn't know whether it's being called directly from the Python interpreter,
or whether it's being used as a component in a larger program.
In the latter case,
the library doesn't know if the program that's calling it is being run from the command line or from a GUI.
The library therefore shouldn't try to handle or report errors itself,
because it has no way of knowing what the right way to do this is.
It should instead just raise an exception,
and let its caller figure out how best to handle it.

Finally,
we can raise exceptions ourselves if we want to.
In fact,
we *should* do this,
since it's the standard way in Python to signal that something has gone wrong.
Here,
for example,
is a function that reads a grid and checks its consistency:


	def read_grid(grid_file):
    	'''Read grid, checking consistency.'''
    	
    	data = read_raw_data(grid_file)
    	if not grid_consistent(data):
        	raise Exception('Inconsistent grid: ' + grid_file)
        result = normalize_grid(data)
    	return result


The `raise` statement creates a new exception with a meaningful error message.
Since `read_grid` itself doesn't contain a `try`/`except` block,
this exception will always be thrown up and out of the function,
to be caught and handled by whoever is calling `read_grid`.
We can define new types of exceptions if we want to.
And we should,
so that errors in our code can be distinguished from errors in other people's code.
However,
this involves classes and objects,
which is outside the scope of these lessons.

### Challenges

Modify the program below so that it prints three lines of output.

    try:
        for number in [-1, 0, 1]:
            print 1.0/number
    except ZeroDivisionError:
        print 'whoops'


## The Limits to Testing

Like any other piece of experimental apparatus,
a complex program requires a much higher investment in testing than a simple one.
Putting it another way,
a small script that is only going to be used once,
to produce one figure,
probably doesn't need separate testing:
its output is either correct or not.
A linear algebra library that will be used by thousands of people
in twice that number of applications
over the course of a decade,
on the other hand,
definitely does.

Unfortunately,
it's practically impossible to prove that a program will always do what it's supposed to.
To see why,
consider a function that checks whether a character strings contains only the letters 'A', 'C', 'G', and 'T'.
These four tests clearly aren't sufficient:


	assert is_all_bases('A')
	assert is_all_bases('C')
	assert is_all_bases('G')
	assert is_all_bases('T')


because this version of `is_all_bases` passes them:


	def is_all_bases(bases):
    	return True


Adding these tests isn't enough:


	assert not is_all_bases('X')
	assert not is_all_bases('Y')
	assert not is_all_bases('Z')


because this version still passes:


	def is_all_bases(bases):
    	return bases[0] in 'ACGT'


We can add yet more tests:


	assert is_all_bases('ACGCGA')
	assert not is_all_bases('CGAZ')


but no matter how many we have,
we can always write a function that passes them,
but does the wrong thing in other cases.
And as we add more tests,
we have to start worrying about whether the tests themselves are correct,
and about whether we can afford the time needed to write them.
After all,
if we really want to check that the square root function is correct for all values between 0.0 and 1.0,
we need to write over a billion test cases;
that's a lot of typing,
and the chances of us getting every one right are effectively zero.

Testing is still worth doing, though:
it's one of those things that doesn't work in theory,
but is surprisingly effective in practice.
If we choose our tests carefully,
we can demonstrate that our software is as likely to be correct as a mathematical proof
or a physical experiment.

Ensuring that we have the right answer is only one reason to to software.
The other is that it speeds up development
by reducing the amount of re-work we have to do.
Even small programs can be quite complex,
and changing one thing can all too easily break something else.
If we test changes as we make them,
and automatically re-test things we've already done,
we can catch and fix errors while the changes are still fresh in our minds.

## Unit Testing

Most people don't enjoy writing tests,
so if we want them to actually do it,
it must be easy to:

- add or change tests,
- understand the tests that have already been written,
- run those tests, and
- understand those tests' results.

Test results must also be reliable.
If a testing tool says that code is working when it's not,
or reports problems when there actually aren't any,
people will lose faith in it and stop using it.

The simplest kind of test is a [unit test](../../gloss.html#unit_test)
that checks the behavior of one component of a program.
As an example,
suppose we're testing a function called `rectangle_area`
that returns the area of an `(x0, y0, x1, y1)` rectangle (you should have this code saved in a file called `rectangle.py`).

    def rectangle_area(coords):
        x0, y0, x1, y1 = coords
        return (x1 - x0) * (x1 - y0)

We'll start by testing our code directly using `assert`.
Here,
we call the function three times with different arguments,
checking that the right value is returned each time.


	from rectangle import rectangle_area

	assert rectangle_area([0, 0, 1, 1]) == 1.0
	assert rectangle_area([1, 1, 4, 4]) == 9.0
	assert rectangle_area([0, 1, 4, 7]) == 24.0


This result is used,
in the sense that we know something's wrong,
but look closely at what happens if we run the tests in a different order:


	assert rectangle_area([0, 1, 4, 7]) == 24.0
	assert rectangle_area([1, 1, 4, 4]) == 9.0
	assert rectangle_area([0, 0, 1, 1]) == 1.0


Python halts at the first failed assertion,
so the second and third tests aren't run at all.
It would be more helpful if we could get data from all of our tests every time they're run,
since the more information we have,
the faster we're likely to be able to track down bugs.
It would also be helpful to have some kind of summary report:
if our [test suite](../../gloss.html#test_suite) includes thirty or forty tests
(as it well might for a complex function or library that's widely used),
we'd like to know how many passed or failed.

Here's a different approach.
First, let's put each test in a function with a meaningful name:


	def test_unit_square():
		assert rectangle_area([0, 0, 1, 1]) == 1.0

	def test_large_square():
		assert rectangle_area([1, 1, 4, 4]) == 9.0

	def test_actual_rectangle():
		assert rectangle_area([0, 1, 4, 7]) == 24.0


Next, let's save this code in a python script (in the same folder as the `rectangle.py` file); name the script `test_rectangle.py`.  Remember to import the `rectangle_area` function! We can now use a test framework for Python called `nose`.

`nose` is a test framework for Python that will automatically find, run and report on tests written in Python. It is an example of what has been termed an *[xUnit test framework](http://en.wikipedia.org/wiki/XUnit)*. The name "xUnit" comes from the fact that
many of them are imitations of a Java testing library called JUnit.
The [Wikipedia page](http://en.wikipedia.org/wiki/List_of_unit_testing_frameworks) on the subject
lists dozens of similar frameworks in almost as many languages,
all of which have a similar structure:
each test is a single function that follows some naming convention
(e.g., starts with `'test_'`),
and the framework runs them in some order
and reports how many passed, failed, or were broken.

To use `nose`, we write test functions, as we've been doing, with the prefix `test_` and put these in files, likewise prefixed by `test_`. The prefixes `Test-`, `Test_` and `test-` can also be used.

To run `nose` for our tests from command line we need to run:

    $ nosetests test_rectangle.py

In the output each `.` corresponds to a successful test. 

### Challenges

1.  A colleague of yours has written a function that calculates the running total of the values in a list,
    e.g.,
    `running([0, 1, 2])` produces the list `[0, 1, 3]`.
    Load this function into your notebook using `from running import running`,
    and then write some unit tests for it using the `ears` library
    to see what bugs you can find.

2.  Some programmers put assertions in their programs to catch errors when they occur;
    others prefer to write unit tests to check that the program is behaving properly.
    Which do you think makes programs easier to read?
    Which do you think makes them easier to maintain as they change over time?

## Test-Driven Development

Libraries like `ear` can't think of test cases for us.
We still have to decide what to test and how many tests to run.
Our best guide here is economics:
we want the tests that are most likely to give us useful information
that we don't already have.
For example,
if `rectangle_area([0, 0, 1, 1])` works,
there's probably not much point testing `rectangle_area((0, 0, 2, 2))`,
since it's hard to think of a bug that would show up in one case but not in the other.

We should therefore try to choose tests that are as different from each other as possible,
so that we force the code we're testing to execute in all the different ways it can.
Another way of thinking about this is that we should try to find [boundary cases](../../gloss.html#boundary_case).
If a function works for zero,
one,
and a million values,
it will probably work for eighteen values.

Using boundary values as tests has another advantage:
it can help us design our software.
To see how,
consider this test case for our rectangle area function:


	def test_inverted_rectangle():
		assert rectangle_area([1, 5, 5, 2]) == -12.0


Is that test correct?
I.e.,
are rectangles with `x1<x0` or `y1<y0` legal,
and do they have negative area?
Or should the test be:


	def test_inverted_rectangle():
		try:
        	rectangle_area([1, 5, 5, 2])
        	assert False, 'Function did not raise exception for invalid rectangle'
        except ValueError:
        	pass # rectangle_area failed with the expected kind of exception
	    except Exception:
	    	assert False, 'Function did not raise correct kind of exception for invalid rectangle'


The logic in this second version may take a moment to work out,
but the idea is straightforward:
we want to check that `rectangle_area` raises a `ValueError` exception
if it's given a rectangle whose upper edge is below or to the left of its lower edge.

Here's another test case that can help us design our software:


	def test_zero_width():
    	assert rectangle_area([2, 1, 2, 8]) == 0


We might decide that rectangles with negative areas aren't allowed,
but what about rectangles with zero area,
i.e.,
rectangles that are actually lines?
Any actual implementation of `rectangle_area` will do *something* with one of these;
writing unit tests for boundary cases is a good way to specify exactly what that something is.

Unit tests are actually such a good way to define how functions ought to behave that
many programmers use a practice called [test-driven development](glossary.html#test_driven_development) (TDD).
Instead of writing code,
then figuring out how to test it,
these programmers:

1. write some unit tests for a function that doesn't exist yet,
2. write that function,
3. modify it until it passes all of the tests, then
4. clean up the function, i.e., make it more readable or more efficient without breaking any of the tests.

The mantra often used during TDD is "[red, green, refactor](../../rules.html#red-green-refactor)":
get a red light (i.e., some failing tests),
make it turn green (i.e., get something working),
and then clean it up by refactoring.
This cycle should take anywhere from a couple of minutes to an hour or so.
If it takes longer than that,
the change being made is probably too large,
and should be broken down into smaller (and more comprehensible) steps.

TDD's proponents argue that it helps people produce better code for two reasons.
First,
it encourages them to write code in small, self-contained chunks,
and to actually write tests for those chunks.
Second,
it frees them from [confirmation bias](../../gloss.html#confirmation_bias):
since they haven't written their function yet,
their subconscious cannot steer their testing toward proving it correct
rather than finding errors.

Empirical studies of TDD have had mixed results:
some have found it beneficial,
while others have found no effect.
But even if you don't use it day to day,
trying it a few times helps you learn how to design functions and programs that are easier to test.

### Challenges

Write a function called `something` that passes the following unit tests:


    def test_empty():
        assert something([]) == []
    
    def test_single_value():
        assert something(['a']) == []
    
    def test_two_values():
        assert something(['a', 'b']) == [('a', 'b')]
    
    def test_three_values():
        assert something(['a', 'b', 'c']) == [('a', 'b'), ('a', 'c'), ('b', 'c')]

