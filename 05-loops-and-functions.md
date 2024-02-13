---
title: Data workflows and automation
teaching: 40
exercises: 50
---

::::::::::::::::::::::::::::::::::::::: objectives

- Describe why for loops are used in Python.
- Employ for loops to automate data analysis.
- Write unique filenames in Python.
- Build reusable code in Python.
- Write functions using conditional statements (if, then, else).

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- Can I automate operations in Python?
- What are functions and why should I use them?

::::::::::::::::::::::::::::::::::::::::::::::::::

So far, we've used Python and the pandas library to explore and manipulate
individual datasets by hand, much like we would do in a spreadsheet. The beauty
of using a programming language like Python, though, comes from the ability to
automate data processing through the use of loops and functions.

## For loops

Loops allow us to repeat a workflow (or series of actions) a given number of
times or while some condition is true. We would use a loop to automatically
process data that's stored in multiple files (daily values with one file per
year, for example). Loops lighten our work load by performing repeated tasks
without our direct involvement and make it less likely that we'll introduce
errors by making mistakes while processing each file by hand.

Let's write a simple for loop that simulates what a kid might see during a
visit to the zoo:

```python
>>> animals = ['lion', 'tiger', 'crocodile', 'vulture', 'hippo']
>>> print(animals)
['lion', 'tiger', 'crocodile', 'vulture', 'hippo']

>>> for creature in animals:
...    print(creature)
lion
tiger
crocodile
vulture
hippo
```

The line defining the loop must start with `for` and end with a colon, and the
body of the loop must be indented.

In this example, `creature` is the loop variable that takes the value of the next
entry in `animals` every time the loop goes around. We can call the loop variable
anything we like. After the loop finishes, the loop variable will still exist
and will have the value of the last entry in the collection:

```python
>>> animals = ['lion', 'tiger', 'crocodile', 'vulture', 'hippo']
>>> for creature in animals:
...    pass

>>> print('The loop variable is now: ' + creature)
The loop variable is now: hippo
```

We are not asking python to print the value of the loop variable anymore, but
the for loop still runs and the value of `creature` changes on each pass through
the loop. The statement `pass` in the body of the loop just means "do nothing".

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge - Loops

1. What happens if we don't include the `pass` statement?

2. Rewrite the loop so that the animals are separated by commas, not new lines
  (Hint: You can concatenate strings using a plus sign. For example,
  `print(string1 + string2)` outputs 'string1string2').
  

::::::::::::::::::::::::::::::::::::::::::::::::::

## Automating data processing using For Loops

Suppose that we were working with a much larger set of books. It might be useful
to split them out into smaller groups, e.g., by date of publication.

Let's start by making a new directory inside our current working folder. Python has
a built-in library called `os` for this sort of operating-system dependent behaviour.

```python
    import os

    os.mkdir('yearly_files')
```

We can check that the folder was created by listing the contents of the current directory:

```python
>>> os.listdir()
['authors.csv', 'yearly_files', 'places.csv', 'eebo.db', 'eebo.csv', '1635.csv', '1640.csv']
```

In previous lessons, we saw how to use the library pandas to load the species
data into memory as a DataFrame, how to select a subset of the data using some
criteria, and how to write the DataFrame into a csv file. Let's write a script
that performs those three steps in sequence to write out records for the year
1636 as a single csv in the `yearly_files` directory

```python
import pandas as pd

# Load the data into a DataFrame
eebo_df = pd.read_csv('eebo.csv')

# Select only data for 1636
authors1636 = eebo_df[eebo_df.Date == "1636"]

# Write the new DataFrame to a csv file
authors1636.to_csv('yearly_files/authors1636.csv')

# Then check that that file now exists:
os.listdir("yearly_files")
```

To create yearly data files, we could repeat the last two commands over and
over, once for each year of data. Repeating code is neither elegant nor
practical, and is very likely to introduce errors into your code. We want to
turn what we've just written into a loop that repeats the last two commands for
every year in the dataset.

Let's start by writing a loop that simply prints the names of the files we want
to create - the dataset we are using covers 1977 through 2002, and we'll create
a separate file for each of those years. Listing the filenames is a good way to
confirm that the loop is behaving as we expect.

We have seen that we can loop over a list of items, so we need a list of years
to loop over. We can get the years in our DataFrame with:

```python
>>> eebo_df['Date']

0      1625
1      1515
2      1528
3      1623
4      1640
5      1623
    ...    
141    1635
142    1614
143    1589
144    1636
145    1562
146    1533
147    1606
148    1618
Name: Date, Length: 149, dtype: int64
```

but we want only unique years, which we can get using the `unique` function
which we have already seen.

```python
>>> eebo_df['Date'].unique()
array([1625, 1515, 1528, 1623, 1640, 1624, 1607, 1558, 1599, 1622, 1613,
       1600, 1635, 1569, 1579, 1597, 1538, 1559, 1563, 1577, 1580, 1626,
       1631, 1565, 1632, 1571, 1554, 1615, 1549, 1567, 1605, 1636, 1591,
       1588, 1619, 1566, 1593, 1547, 1603, 1609, 1589, 1574, 1584, 1630,
       1621, 1610, 1542, 1534, 1519, 1550, 1540, 1557, 1606, 1545, 1537,
       1532, 1526, 1531, 1533, 1572, 1536, 1529, 1535, 1543, 1586, 1596,
       1552, 1608, 1611, 1616, 1581, 1639, 1570, 1564, 1568, 1602, 1618,
       1583, 1638, 1592, 1544, 1585, 1614, 1562])
```

Putting this into our for loop we get

```python
>>> for year in eebo_df['Date'].unique():
...    filename = 'yearly_files/authors{}.csv'.format(year)
...    print(filename)
...
yearly_files/authors1625.csv
yearly_files/authors1515.csv
yearly_files/authors1528.csv
yearly_files/authors1623.csv
yearly_files/authors1640.csv
yearly_files/authors1624.csv
yearly_files/authors1607.csv
```

We can now add the rest of the steps we need to create separate text files:

```python
# Load the data into a DataFrame
eebo_df = pd.read_csv('data/eebo.csv')

for year in eebo_df['Date'].unique():

    # Select data for the year
    publish_year = eebo_df[eebo_df.Date == year]

    # Write the new DataFrame to a csv file
    filename = 'yearly_files/authors{}.csv'.format(year)
    publish_year.to_csv(filename)
```

Look inside the `yearly_files` directory and check a couple of the files you
just created to confirm that everything worked as expected.

## Writing Unique Filenames

Notice that the code above created a unique filename for each year.

```
filename = 'yearly_files/authors{}.csv'.format(year)
```

Let's break down the parts of this name:

- The first part is simply some text that specifies the directory to store our
  data file in `yearly_files/` and the first part of the file name
  (authors): `'yearly_files/authors'`
- We want to dynamically insert the value of the year into the filename. We can
  do this by indicating a placeholder location inside the string with `{}`, and
  then specifying the value that will go there with `.format(value)`
- Finally, we specify a file type with `.csv`. Since the `.` is inside the
  string, it doesn't behave the same way as dot notation in Python commands.

Notice the difference between the filename - wrapped in quote marks - and the
variable (`year`), which is not wrapped in quote marks. The result looks like
`'yearly_files/authors1607.csv'` which contains the path to the new filename
AND the file name itself.

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge - Modifying loops

1. Some of the surveys you saved are missing data (they have null values that
  show up as NaN - Not A Number - in the DataFrames and do not show up in the text
  files). Modify the for loop so that the entries with null values are not
  included in the yearly files.

2. What happens if there is no data for a year?

3. Instead of splitting out the data by years, a colleague wants to analyse each place separately. How would you write a unique csv file for each location?
  

::::::::::::::::::::::::::::::::::::::::::::::::::

## Building reusable and modular code with functions

Suppose that separating large data files into individual yearly files is a task
that we frequently have to perform. We could write a **for loop** like the one above
every time we needed to do it but that would be time consuming and error prone.
A more elegant solution would be to create a reusable tool that performs this
task with minimum input from the user. To do this, we are going to turn the code
we've already written into a function.

Functions are reusable, self-contained pieces of code that are called with a
single command. They can be designed to accept arguments as input and return
values, but they don't need to do either. Variables declared inside functions
only exist while the function is running and if a variable within the function
(a local variable) has the same name as a variable somewhere else in the code,
the local variable hides but doesn't overwrite the other.

Every method used in Python (for example, `print`) is a function, and the
libraries we import (say, `pandas`) are a collection of functions. We will only
use functions that are housed within the same code that uses them, but it's also
easy to write functions that can be used by different programs.

Functions are declared following this general structure:

```python
def this_is_the_function_name(input_argument1, input_argument2):
    # The body of the function is indented
    """This is the docstring of the function. Wrapped in triple-quotes,
    it can span across multiple lines. This is what is shown if you ask
    for help about the function like this:
    >>> help(this_is_the_function_name)
    """
    
    # This function prints the two arguments to screen
    print('The function arguments are:', input_argument1, input_argument2, '(this is done inside the function!)')

    # And returns their product
    return input_argument1 * input_argument2
```

The function declaration starts with the word `def`, followed by the function
name and any arguments in parenthesis, and ends in a colon. The body of the
function is indented just like loops are. If the function returns something when
it is called, it includes a return statement at the end.

This is how we call the function:

```python
>>> product_of_inputs = this_is_the_function_name(2,5)
The function arguments are: 2 5 (this is done inside the function!)

>>> print('Their product is:', product_of_inputs, '(this is done outside the function!)')
Their product is: 10 (this is done outside the function!)
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge - Functions

1. Change the values of the arguments in the function and check its output
2. Try calling the function by giving it the wrong number of arguments (not 2)
  or not assigning the function call to a variable (no `product_of_inputs =`)
3. Declare a variable inside the function and test to see where it exists (Hint:
  can you print it from outside the function?)
4. Explore what happens when a variable both inside and outside the function
  have the same name. What happens to the global variable when you change the
  value of the local variable?
  

::::::::::::::::::::::::::::::::::::::::::::::::::

We can now turn our code for saving yearly data files into a function. There are
many different "chunks" of this code that we can turn into functions, and we can
even create functions that call other functions inside them. Let's first write a
function that separates data for just one year and saves that data to a file:

```python
def one_year_csv_writer(this_year, all_data):
    """
    Writes a csv file for data from a given year.
    
    Parameters
    ----------
    this_year: int
        year for which data is extracted
    all_data: pandas Dataframe
        DataFrame with multi-year data
    
    Returns
    -------
    None
    """

    # Select data for the year
    texts_year = all_data[all_data.Date == this_year]

    # Write the new DataFrame to a csv file
    filename = 'data/yearly_files/function_authors' + str(this_year) + '.csv'
    texts_year.to_csv(filename)
```

The text between the two sets of triple quotes is called a docstring and
contains the documentation for the function. It does nothing when the function
is running and is therefore not necessary, but it is good practice to include
docstrings as a reminder of what the code does. Docstrings in functions also
become part of their 'official' documentation:

```python
one_year_csv_writer?
```

```python
one_year_csv_writer('1607',eebo_df)
```

We changed the root of the name of the csv file so we can distinguish it from
the one we wrote before. Check the `yearly_files` directory for the file. Did it
do what you expect?

What we really want to do, though, is create files for multiple years without
having to request them one by one. Let's write another function that replaces
the entire For loop by simply looping through a sequence of years and repeatedly
calling the function we just wrote, `one_year_csv_writer`:

```python
def yearly_data_csv_writer(start_year, end_year, all_data):
    """
    Writes separate csv files for each year of data.

    Parameters
    ----------
    start_year: int
        the first year of data we want
    end_year: int
        the last year of data we want
    all_data: pandas Dataframe
        DataFrame with multi-year data

    Returns
    -------
    None
    """

    # "end_year" is the last year of data we want to pull, so we loop to end_year+1
    for year in range(start_year, end_year+1):
        one_year_csv_writer(str(year), all_data)
```

Because people will naturally expect that the end year for the files is the last
year with data, the for loop inside the function ends at `end_year + 1`. By
writing the entire loop into a function, we've made a reusable tool for whenever
we need to break a large data file into yearly files. Because we can specify the
first and last year for which we want files, we can even use this function to
create files for a subset of the years available. This is how we call this
function:

```python
# Load the data into a DataFrame
eebo_df = pd.read_csv('data/eebo.csv')

# Create csv files
yearly_data_csv_writer(1500, 1650, eebo_df)
```

BEWARE! If you are using IPython Notebooks and you modify a function, you MUST
re-run that cell in order for the changed function to be available to the rest
of the code. Nothing will visibly happen when you do this, though, because
simply defining a function without *calling* it doesn't produce an output. Any
cells that use the now-changed functions will also have to be re-run for their
output to change.

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge- More functions

1. Add two arguments to the functions we wrote that take the path of the
  directory where the files will be written and the root of the file name.
  Create a new set of files with a different name in a different directory.
2. How could you use the function `yearly_data_csv_writer` to create a csv file
  for only one year? (Hint: think about the syntax for `range`)
3. Make the functions return a list of the files they have written. There are
  many ways you can do this (and you should try them all!): either of the
  functions can print to screen, either can use a return statement to give back
  numbers or strings to their function call, or you can use some combination of
  the two. You could also try using the `os` library to list the contents of
  directories.
4. Explore what happens when variables are declared inside each of the functions
  versus in the main (non-indented) body of your code. What is the scope of the
  variables (where are they visible)? What happens when they have the same name
  but are given different values?
  

::::::::::::::::::::::::::::::::::::::::::::::::::

The functions we wrote demand that we give them a value for every argument.
Ideally, we would like these functions to be as flexible and independent as
possible. Let's modify the function `yearly_data_csv_writer` so that the
`start_year` and `end_year` default to the full range of the data if they are
not supplied by the user. Arguments can be given default values with an equal
sign in the function declaration. Any arguments in the function without default
values (here, `all_data`) is a required argument and MUST come before the
argument with default values (which are optional in the function call).

```python
def yearly_data_arg_test(all_data, start_year=1000, end_year=2000):
    """
    Test function with default values. Doesn't actually do anything
    except return the arguments, to check what they end up as

    Parameters
    ----------
    all_data: pandas Dataframe
        DataFrame with multi-year data
    start_year: int, optional
        the first year of data we want
        Default 1000
    end_year: int, optional
        the last year of data we want

    Returns
    -------
    int
        start_year
    int: 
        end_year
    """

    return start_year, end_year


start,end = yearly_data_arg_test (eebo_df, 1600, 1660)
print('Both optional arguments:\t', start, end)

start,end = yearly_data_arg_test (eebo_df)
print('Default values:\t\t\t', start, end)
```

```
Both optional arguments:	1600 1660
Default values:			    1000 2000
```

The "\\t" in the `print` statements are tabs, used to make the text align and be
easier to read.

But what if our dataset doesn't start in 1300 and end in 1700? We can modify the
function so that it looks for the start and end years in the dataset if those
dates are not provided:

```python
def yearly_data_arg_test(all_data, start_year=None, end_year=None):
    """
    Test function with default values. Doesn't actually do anything
    except return the arguments, to check what they end up as

    Parameters
    ----------
    all_data: pandas Dataframe
        DataFrame with multi-year data
    start_year: int, optional
        the first year of data we want.
        Defaults to earliest year in `all_data`
    end_year: int, optional
        the last year of data we want
        Defaults to latest year in `all_data`

    Returns
    -------
    int
        start_year
    int: 
        end_year
    """

    if start_year is None:
        start_year = min(all_data.Date)
    if end_year is None:
        end_year = max(all_data.Date)

    return start_year, end_year


start,end = yearly_data_arg_test (eebo_df, '1600', '1660')
print('Both optional arguments:\t', start, end)

start,end = yearly_data_arg_test (eebo_df)
print('Default values:\t\t\t', start, end)
```

```
Both optional arguments:	1600 1660
Default values:		        1500 1650
```

The default values of the `start_year` and `end_year` arguments in the function
`yearly_data_arg_test` are now `None`. This is a build-in constant in Python
that indicates the absence of a value - essentially, that the variable exists in
the namespace of the function (the directory of variable names) but that it
doesn't correspond to any existing object.

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge - Variables

1. What type of object corresponds to a variable declared as `None`? (Hint:
  create a variable set to `None` and use the function `type()`)

2. Compare the behavior of the function `yearly_data_arg_test` when the
  arguments have `None` as a default and when they do not have default values.

3. What happens if you only include a value for `start_year` in the function
  call? Can you write the function call with only a value for `end_year`? (Hint:
  think about how the function must be assigning values to each of the arguments -
  this is related to the need to put the arguments without default values before
  those with default values in the function definition!)
  

::::::::::::::::::::::::::::::::::::::::::::::::::

## If Loops

The body of the test function now has two conditional loops (if loops) that
check the values of `start_year` and `end_year`. If loops execute the body of
the loop when some condition is met. They commonly look something like this:

```python
a = 5

if a<0: # meets first condition?

    # if a IS less than zero
    print('a is a negative number')

elif a>0: # did not meet first condition. meets second condition?

    # if a ISN'T less than zero and IS more than zero
    print('a is a positive number')

else: # met neither condition

    # if a ISN'T less than zero and ISN'T more than zero
    print('a must be zero!')
```

Which would return:

```
a is a positive number
```

Change the value of `a` to see how this function works. The statement `elif`
means "else if", and all of the conditional statements must end in a colon.

The if loops in the function `yearly_data_arg_test` check whether there is an
object associated with the variable names `start_year` and `end_year`. If those
variables are `None`, the if loops return the boolean `True` and execute whaever
is in their body. On the other hand, if the variable names are associated with
some value (they got a number in the function call), the if loops return `False`
and do not execute. The opposite conditional statements, which would return
`True` if the variables were associated with objects (if they had received value
in the function call), would be `if start_year` and `if end_year`.

As we've written it so far, the function `yearly_data_arg_test` associates
values in the function call with arguments in the function definition just based
in their order. If the function gets only two values in the function call, the
first one will be associated with `all_data` and the second with `start_year`,
regardless of what we intended them to be. We can get around this problem by
calling the function using keyword arguments, where each of the arguments in the
function definition is associated with a keyword and the function call passes
values to the function using these keywords:

```python

start,end = yearly_data_arg_test (eebo_df)
print('Default values:\t\t\t', start, end)

start,end = yearly_data_arg_test (eebo_df, 1600, 1660)
print('No keywords:\t\t\t', start, end)

start,end = yearly_data_arg_test (eebo_df, start_year=1600, end_year=1660)
print('Both keywords, in order:\t', start, end)

start,end = yearly_data_arg_test (eebo_df, end_year=1660, start_year=1600)
print('Both keywords, flipped:\t\t', start, end)

start,end = yearly_data_arg_test (eebo_df, start_year=1600)
print('One keyword, default end:\t', start, end)

start,end = yearly_data_arg_test (eebo_df, end_year=1660)
print('One keyword, default start:\t', start, end)
```

```
Default values:			1515 1640
No keywords:			1600 1640
Both keywords, in order:	        1600 1660
Both keywords, flipped:		1600 1660
One keyword, default end:	        1600 1640
One keyword, default start:	        1515 1660
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Challenge - Modifying functions

1. Rewrite the `one_year_csv_writer` and `yearly_data_csv_writer` functions to
  have keyword arguments with default values

2. Modify the functions so that they don't create yearly files if there is no
  data for a given year and display an alert to the user (Hint: use conditional
  statements and if loops to do this. For an extra challenge, use `try`
  statements!)

3. The code below checks to see whether a directory exists and creates one if it
  doesn't. Add some code to your function that writes out the CSV files, to check
  for a directory to write to.

```Python
  if 'dir_name_here' in os.listdir('.'):
      print('Processed directory exists')
  else:
      os.mkdir('dir_name_here')
      print('Processed directory created')
```

4. The code that you have written so far to loop through the years is good,
  however it is not necessarily reproducible with different datasets.
  For instance, what happens to the code if we have additional years of data
  in our CSV files? Using the tools that you learned in the previous activities,
  make a list of all years represented in the data. Then create a loop to process
  your data, that begins at the earliest year and ends at the latest year using
  that list.

HINT: you can create a loop with a list as follows: `for years in year_list:`


::::::::::::::::::::::::::::::::::::::::::::::::::


