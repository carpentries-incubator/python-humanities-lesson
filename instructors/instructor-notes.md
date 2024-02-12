---
title: Instructor Notes
permalink: /guide/
---

# Challenge solutions

## Checking installations.

In the include/scripts directory, you will find a script called check\_env.py This checks the functionality of the Anaconda install.

By default, Data Carpentry does not have people pull the whole repository with all the scripts and addenda. Therefore,
you, as the instructor, get to decide how you'd like to provide this script to learners, if at all.
To use this, students can navigate into includes/scripts terminal, and execute the following:

```
python check_env.py 
```

If learners receive an `AssertionError`, it will inform you how to help them correct this
installation. Otherwise, it will tell you that the system is good to go and ready for Data Carpentry!

## 00-short-introduction-to-Python

### Tuples Challenges

- What happens when you type `a_tuple[2] = 5` vs `a_list[1] = 5`?
  
  As a tuple is immutable, it does not support item assignment. Elements in a list can be altered individually.

- Type `type(a_tuple)` into the Python interpreter - what is the object type?
  
  `tuple`

### Dictionaries Challenges

- Can you do reassignment in a dictionary? Give it a try.

Make sure it is also clear that access to 'the second value' is actually just about the key name. Add for example `rev[10] = "ten"` to clarify it is not about the position.

- You should see the following output:
  `{1: 'one', 2: 'two', 3: 'three'}`

```python
rev[2] = "apple-sauce"
```

```python
{1: 'one', 2: 'apple-sauce', 3: 'three'}
```

## 01-starting-with-data

\###Bug Note:

Pandas \< .18.1 has a bug where surveys\_df['weight'].describe() may return a runtime error. This refers to the old ecology data, not clear if it applies to the eebo data.

### Dataframe Challenges

- `eebo_df.columns`

- `eebo_df.head()`. Also, what does `eebo_df.head(15)` do?
  
  Show first `N` lines

- `eebo_df.tail()`
  
  Show last `N` lines

- `eebo_df.shape`. Take note of the output of the shape method. What format does it return the shape of the DataFrame in?
  
  `type(surveys_df.shape)` -> `Tuple`, (rows, columns), i.e. standard row-first Python format

### Calculating Statistics Challenges

- Create a list of unique locations found in the index data. Call it `places`. How many unique location are there in the data?
  
  `places = pd.unique(eebo_df["Place"])`
  `len(places)`

- What is the difference between `len(places)` and `eebo_df["Place"].nunique()`?

Both do result in the same output, making it alternative ways of getting the unique values. `nunique` combines the count and unique value extraction.

### Plotting Challenges

- Create a plot of Authors across all Places per plot.
  
  - This requires several steps in the correct order. The easiest one to miss is the .unstack() command
  
  ```python
  date_place = eebo_df.groupby(['Date','Place'])
  page_sum = date_place['Page Count'].sum()
  table = page_sum.unstack()
  plot = table.plot(kind="bar", stacked=True, title="Pages published per year")
  plot.set_ylabel("Pages")
  ```
  
  - Since the list of places is long, the legend tends to wind up hiding half the graph. It's well worth specifying an increased figure size to compensate. Remember that the figsize is interpreted in inches assuming 96dpi, i.e. `figsize=(16,8)` means 16 inches wide and 8 inches high

## 02-index-slice-subset

### Indexing Challenges

- What value does the code below return? `a[0]`
  
  `1`, as  Python starts with element 0 (for Matlab users: this is different!)

- How about this: `a[5]`
  
  `IndexError`

- In the example above, calling `a[5]` returns an error. Why is that?
  
  The list has no element with index 5 (going from 0 till 4).

- What about? `a[len(a)]`
  
  `IndexError`

### Selection Challenges

- What happens when you type:
  
  `eebo_df[0:3]`
  `eebo_df[:5]` slicing from first element makes 0 redundant
  `eebo_df[-1:]` you can count backwards
  
  *Suggestion*: You can also select every Nth row: `eebo_df[1:10:2]`. So, how to interpret `eebo_df[::-1]`?

- What is the difference between `eebo_df.iloc[0:4, 1:4]` and `eebo_df.loc[0:4, 1:4]`?
  
  Check the position, or the name. Cfr. the second is like it would be in a dictionary, asking for the key-names. Column names 1:4 do not exist, resulting in an error. Check also the difference between `eebo_df.loc[0:4]` and `eebo_df.iloc[0:4]`

Questions seem to arise around queries in the form of \`eebo\_df[eebo\_df[...]], i.e. compound indexing in a single line. It may be helpful to explain the order in which these statements are evaluated by splitting them into multiple lines.

### Advanced Queries Challenges

- Select a subset of rows in the `eebo_df` DataFrame that contain data from the year 1540 and that contain page count values less than or equal to 8. How many rows did you end up with? What did your neighbor get?
  
  `eebo_df[ (eebo_df["Date"] <= 1540) & (eebo_df["Page Count"] <= 8) ]`
  Should be 2 values

- Use the `isin` function to find all books written by Robert Aylett and Robert Aytoun. How many are there?
  
  `eebo_df[ eebo_df["Author"].isin(["Aylett, Robert, 1583-1655?", "Aytoun, Robert, Sir, 1570-1638."]) ]`
  7

- Write a query that selects all rows with Date NOT equal to 1500 or 1600 in the eebo data.
  
  `eebo_df[ ~eebo_df["Date"].isin([1500, 1600]) ]`
  147

### Masking Challenges

Since the data is quite ugly, the `contains` method will come in handy

- Create a new DataFrame that only contains titles with status values that are not from London. Assign each status value in the new DataFrame to a new value of ‘x'. Determine the number of null values in the subset.
  
  `x = eebo_df[(~eebo_df["Place"].str.contains("London", na=False)) & (~pd.isnull(eebo_df["Status"]))]`
  No Status values are null ( `pd.isnull(x["Status"]).any()` )

- Create a new DataFrame that contains only observations that are of status free and where page count values are greater than 100.
  
  `eebo_df[(eebo_df["Status"] == "Free") & (eebo_df["Page Count"] > 100)]`
  41
  SInce every single record has a status of `Free`, this is a bit of a trivial question, but there we go

## 03-data-types-and-format

### Challenge - Counting

eebo\_df.isnull()

## 04-merging-data

### Combine Data Challenge

- In the data folder, there are two catalogue data files: 1635.csv and 1640.csv. Read the data into python and combine the files to make one new data frame.

```python
csv_1 = pd.read_csv("1635.csv")
csv_2 = pd.read_csv("1640.csv")
combined = pd.concat( [csv_1, csv_2], axis=0).reset_index(drop=True)
```

### Distributions Challenge

FIXME

## 05-loops-and-functions

### Basic Loop Challenges

- What happens if we do not include the `pass` statement?
  
  `SyntaxError:`

- Rewrite the loop so that the animals are separated by commas, not new lines (Hint: You can concatenate strings using a plus sign. For example, `print(string1 + string2)` outputs 'string1string2').

```python
for creature in animals:
    print(creature+',', end='')
```

This loop also adds a comma after the last animal. A better, loop-free solution would be: `','.join(animals)`

### Looping Over Dataframe Challenges

- Some of the surveys you saved are missing data (they have null values that show up as NaN - Not A Number - in the DataFrames and do not show up in the text files). Modify the for loop so that the entries with null values are not included in the yearly files.

```python
eebo_year = eebo_df[eebo_df.year == year].dropna()
```

- What happens if there is no data for a year?
  
  Nothing really. Since we are explicitly looping over the years where there *are* data, this is a deliberately out-of-context question - but it's useful to see whether people recognise that, or get confused.

- Instead of splitting out the data by years, a colleague wants to analyse each place separately. How would you write a unique csv file for each location?
  
  Similar to previous example, but use the `Place` column. `eebo_df['Place'].unique()`.

### Functions Challenges

- Change the values of the arguments in the function and check its output.

- Try calling the function by giving it the wrong number of arguments (not 2), or not assigning the function call to a variable (no `product_of_inputs =`).

- Declare a variable inside the function and test to see where it exists (Hint: can you print it from outside the function?).

- Explore what happens when a variable both inside and outside the function have the same name. What happens to the global variable when you change the value of the local variable?
  
  Show these in a debugging environment to make this more clear!

### Additional Functions Challenges

- Add two arguments to the functions we wrote that take the path of the directory where the files will be written and the root of the file name. Create a new set of files with a different name in a different directory.

```python
def one_year_csv_writer(this_year, all_data, folder_to_save, root_name):
    """
    Writes a csv file for data from a given year.

    Parameters
    ---------
    this_year : int
        year for which data is extracted
    all_data: pd.DataFrame
        DataFrame with multi-year data 
    folder_to_save : str
        folder to save the data files
    root_name: str
        root of the filenames to save the data
    """

    # Select data for the year
    surveys_year = all_data[all_data.year == this_year]

    # Write the new DataFrame to a csv file
    filename = os.path.join(folder_to_save, ''.join([root_name, str(this_year), '.csv']))
    surveys_year.to_csv(filename)
```

Also adapt function `yearly_data_csv_writer` with the additional inputs.

### Additional Functions Challenges

- How could you use the function `yearly_data_csv_writer` to create a csv file for only one year? (Hint: think about the syntax for `range`)
  
  Adapt the input arguments, e.g. 1978, 1979.

### Output Management Challenges

- Make the functions return a list of the files they have written. There are many ways you can do this (and you should try them all!):
  - either of the functions can print to screen,
    just add `print("year " + str(this_year)+ " written to disk")` statement
  - either can use a return statement to give back numbers or strings to their function call,
  - or you can use some combination of the two.
  - You could also try using the os library to list the contents of directories.
    `os.listdir`

Implementation inside the function:

```python
filenames = []
for year in range(start_year, end_year+1):
    filenames.append(one_year_csv_writer(year, all_data, folder_to_save, root_name))
return filenames
```

- Explore what happens when variables are declared inside each of the functions versus in the main (non-indented) body of your code. What is the scope of the variables (where are they visible)? What happens when they have the same name but are given different values?

- What type of object corresponds to a variable declared as `None`? (Hint: create a variable set to `None` and use the function `type()`)
  
  `NoneType`

- Compare the behavior of the function `yearly_data_arg_test` when the arguments have `None` as a default and when they do not have default values.

- What happens if you only include a value for `start_year` in the function call? Can you write the function call with only a value for `end_year`? (Hint: think about how the function must be assigning values to each of the arguments - this is related to the need to put the arguments without default values before those with default values in the function definition!)

```python
yearly_data_arg_test(surveys_df, end_year=2001)
```

- Rewrite the `one_year_csv_writer` and `yearly_data_csv_writer` functions to have keyword arguments with default values.

```python
def one_year_csv_writer(this_year, all_data, folder_to_save='./', root_name='survey'):
    """
    Writes a csv file for data from a given year.

    Parameters
    ---------
    this_year : int
        year for which data is extracted
    all_data: pd.DataFrame
        DataFrame with multi-year data 
    folder_to_save : str
        folder to save the data files
    root_name: str
        root of the filenames to save the data
    """

    # Select data for the year
    surveys_year = all_data[all_data.year == this_year]

    # Write the new DataFrame to a csv file
    filename = os.path.join(folder_to_save, ''.join([root_name, str(this_year), '.csv']))
    surveys_year.to_csv(filename)
```

- Modify the functions so that they do not create yearly files if there is no data for a given year and display an alert to the user (Hint: use `for` loops and conditional statements to do this. For an extra challenge, use `try` statements!)

```python
    # Write the new DataFrame to a csv file
    if len(surveys_year) > 0:
        filename = os.path.join(folder_to_save, ''.join([root_name, str(this_year), '.csv']))
        surveys_year.to_csv(filename)
    else:
        print("No data for year " + str(this_year))
```

- The code that you have written so far to loop through the years is good, however, it is not necessarily reproducible with different datasets. For instance, what happens to the code if we have additional years of data in our CSV files? Using the tools that you learned in the previous activities, make a list of all years represented in the data. Then create a loop to process your data, that begins at the earliest year and ends at the latest year using that list.

```python
def yearly_data_csv_writer(all_data, yearcolumn="year", 
                           folder_to_save='./', root_name='survey'):
    """
    Writes separate csv files for each year of data.

    all_data --- DataFrame with multi-year data
    yearcolumn --- column name containing the year of the data
    folder_to_save --- folder name to store files
    root_name --- start of the file names stored
    """
    years = all_data["year"].unique()

    # "end_year" is the last year of data we want to pull, so we loop to end_year+1
    filenames = []
    for year in years:
        filenames.append(one_year_csv_writer(year, all_data, folder_to_save, root_name))
    return filenames
```


