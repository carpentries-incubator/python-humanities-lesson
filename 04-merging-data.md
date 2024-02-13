---
title: Combining DataFrames with pandas
teaching: 20
exercises: 25
---

::::::::::::::::::::::::::::::::::::::: objectives

- Combine data from multiple files into a single DataFrame using merge and concat.
- Combine two DataFrames using a unique ID found in both DataFrames.
- Employ `to_csv` to export a DataFrame in CSV format.
- Join DataFrames using common fields (join keys).

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- Can I work with data from multiple sources?
- How can I combine data from different data sets?

::::::::::::::::::::::::::::::::::::::::::::::::::

In many "real world" situations, the data that we want to use come in multiple
files. We often need to combine these files into a single DataFrame to analyze
the data. The pandas package provides [various methods for combining
DataFrames](https://pandas.pydata.org/pandas-docs/stable/merging.html) including
`merge` and `concat`.

To work through the examples below, we first need to load the species and
surveys files into pandas DataFrames. The authors.csv and places.csv data can be found in the data folder.

```python
import pandas as pd
authors_df = pd.read_csv("authors.csv",
                         keep_default_na=False, na_values=[""])
authors_df

        TCP                                             Author
0    A00002                         Aylett, Robert, 1583-1655?
1    A00005  Higden, Ranulf, d. 1364. Polycronicon. English...
2    A00007             Higden, Ranulf, d. 1364. Polycronicon.
3    A00008          Wood, William, fl. 1623, attributed name.
4    A00011

places_df = pd.read_csv("places.csv",
                         keep_default_na=False, na_values=[""])
places_df
    A00002                         London
0   A00005                         London
1   A00007                         London
2   A00008               The Netherlands?
3   A00011                      Amsterdam
4   A00012                         London
5   A00014                         London

```

Take note that the `read_csv` method we used can take some additional options which
we didn't use previously. Many functions in python have a set of options that
can be set by the user if needed. In this case, we have told Pandas to assign
empty values in our CSV to NaN `keep_default_na=False, na_values=[""]`.
[More about all of the read\_csv options here.](https://pandas.pydata.org/pandas-docs/dev/generated/pandas.io.parsers.read_csv.html)

## Concatenating DataFrames

We can use the `concat` function in Pandas to append either columns or rows from
one DataFrame to another.  Let's grab two subsets of our data to see how this
works.

```python
# read in first 10 lines of the places table
place_sub = places_df.head(10)
# grab the last 20 rows 
place_sub_last10 = places_df.tail(20)
#reset the index values to the second dataframe appends properly
place_sub_last10 = place_sub_last10.reset_index(drop=True)
# drop=True option avoids adding new index column with old index values
```

When we concatenate DataFrames, we need to specify the axis. `axis=0` tells
Pandas to stack the second DataFrame under the first one. It will automatically
detect whether the column names are the same and will stack accordingly.
`axis=1` will stack the columns in the second DataFrame to the RIGHT of the
first DataFrame. To stack the data vertically, we need to make sure we have the
same columns and associated column format in both datasets. When we stack
horizonally, we want to make sure what we are doing makes sense (ie the data are
related in some way).

```python
# stack the DataFrames on top of each other
vertical_stack = pd.concat([place_sub, place_sub_last10], axis=0)

# place the DataFrames side by side
horizontal_stack = pd.concat([place_sub, place_sub_last10], axis=1)
```

#### Row Index Values and Concat

Have a look at the `vertical_stack` dataframe? Notice anything unusual?
The row indexes for the two data frames `place_sub` and `place_sub_last10`
have been repeated. We can reindex the new dataframe using the `reset_index()` method.

### Writing Out Data to CSV

We can use the `to_csv` command to do export a DataFrame in CSV format. Note that the code
below will by default save the data into the current working directory. We can
save it to a different folder by adding the foldername and a slash to the file
`vertical_stack.to_csv('foldername/out.csv')`. We use the 'index=False' so that
pandas doesn't include the index number for each line.

```python
# Write DataFrame to CSV
vertical_stack.to_csv('out.csv', index=False)
```

Check out your working directory to make sure the CSV wrote out properly, and
that you can open it! If you want, try to bring it back into python to make sure
it imports properly.

```python
# for kicks read our output back into python and make sure all looks good
new_output = pd.read_csv('out.csv', keep_default_na=False, na_values=[""])
```

:::::::::::::::::::::::::::::::::::::::  challenge

### Challenge - Combine Data

In the data folder, there are two catalogue data files: `1635.csv` and
`1640.csv`. Read the data into python and combine the files to make one
new data frame.

:::::::::::::::  solution

### Solution to Challenge

```python
 csv_1 = pd.read_csv("1635.csv")
 csv_2 = pd.read_csv("1640.csv")
 combined = pd.concat( [csv_1, csv_2], axis=0).reset_index(drop=True)
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Joining DataFrames

When we concatenated our DataFrames we simply added them to each other -
stacking them either vertically or side by side. Another way to combine
DataFrames is to use columns in each dataset that contain common values (a
common unique id). Combining DataFrames using a common field is called
"joining". The columns containing the common values are called "join key(s)".
Joining DataFrames in this way is often useful when one DataFrame is a "lookup
table" containing additional data that we want to include in the other.

NOTE: This process of joining tables is similar to what we do with tables in an
SQL database.

The `places.csv` file is table that contains the place and EEBO id for some titles.
When we want to access that information, we can create a query that joins the additional
columns of information to the author data.

Storing data in this way has many benefits including:

### Identifying join keys

To identify appropriate join keys we first need to know which field(s) are
shared between the files (DataFrames). We might inspect both DataFrames to
identify these columns. If we are lucky, both DataFrames will have columns with
the same name that also contain the same data. If we are less lucky, we need to
identify a (differently-named) column in each DataFrame that contains the same
information.

```python
>>> authors_df.columns

Index(['TCP', 'Author'], dtype='object')

>>> places_df.columns

Index(['TCP', 'Place'], dtype='object')

```

In our example, the join key is the column containing the identifier, which is called `TCP`.

Now that we know the fields with the common TCP ID attributes in each
DataFrame, we are almost ready to join our data. However, since there are
[different types of joins](https://blog.codinghorror.com/a-visual-explanation-of-sql-joins/), we
also need to decide which type of join makes sense for our analysis.

### Inner joins

The most common type of join is called an *inner join*. An inner join combines
two DataFrames based on a join key and returns a new DataFrame that contains
**only** those rows that have matching values in *both* of the original
DataFrames.

Inner joins yield a DataFrame that contains only rows where the value being
joins exists in BOTH tables. An example of an inner join, adapted from [this
page](https://blog.codinghorror.com/a-visual-explanation-of-sql-joins/) is below:

![](https://blog.codinghorror.com/content/fig/uploads/2007/10/6a0120a85dcdae970b012877702708970c-pi.png){alt='Inner join -- courtesy of codinghorror.com'}

The pandas function for performing joins is called `merge` and an Inner join is
the default option:

```python
merged_inner = pd.merge(left=authors_df,right=places_df, left_on='TCP', right_on='TCP')
# in this case `species_id` is the only column name in  both dataframes, so if we skippd `left_on`
# and `right_on` arguments we would still get the same result

# what's the size of the output data?
merged_inner.shape
merged_inner
```

**OUTPUT:**

```
      TCP                                             Author             Place
0  A00002                         Aylett, Robert, 1583-1655?            London
1  A00005  Higden, Ranulf, d. 1364. Polycronicon. English...            London
2  A00007             Higden, Ranulf, d. 1364. Polycronicon.            London
3  A00008          Wood, William, fl. 1623, attributed name.  The Netherlands?
4  A00011                                                NaN         Amsterdam
```

The result of an inner join of `authors_df` and `places_df` is a new DataFrame
that contains the combined set of columns from those tables. It
*only* contains rows that have two-letter species codes that are the same in
both the `authos_df` and `place_df` DataFrames. In other words, if a row in
`authors_df` has a value of `TCP` that does *not* appear in the `TCP`
column of `TCP`, it will not be included in the DataFrame returned by an
inner join.  Similarly, if a row in `places_df` has a value of `TCP`
that does *not* appear in the `TCP` column of `places_df`, that row will not
be included in the DataFrame returned by an inner join.

The two DataFrames that we want to join are passed to the `merge` function using
the `left` and `right` argument. The `left_on='TCP'` argument tells `merge`
to use the `TCP` column as the join key from `places_df` (the `left`
DataFrame). Similarly , the `right_on='TCP'` argument tells `merge` to
use the `TCP` column as the join key from `authors_df` (the `right`
DataFrame). For inner joins, the order of the `left` and `right` arguments does
not matter.

The result `merged_inner` DataFrame contains all of the columns from `authors`
(TCP, Person) as well as all the columns from `places_df`
(TCP, Place).

Notice that `merged_inner` has fewer rows than `place_sub`. This is an
indication that there were rows in `place_df` with value(s) for `EEBO` that
do not exist as value(s) for `EEBO` in `authors_df`.

### Left joins

What if we want to add information from `cat_sub` to `survey_sub` without
losing any of the information from `survey_sub`? In this case, we use a different
type of join called a "left outer join", or a "left join".

Like an inner join, a left join uses join keys to combine two DataFrames. Unlike
an inner join, a left join will return *all* of the rows from the `left`
DataFrame, even those rows whose join key(s) do not have values in the `right`
DataFrame.  Rows in the `left` DataFrame that are missing values for the join
key(s) in the `right` DataFrame will simply have null (i.e., NaN or None) values
for those columns in the resulting joined DataFrame.

Note: a left join will still discard rows from the `right` DataFrame that do not
have values for the join key(s) in the `left` DataFrame.

![](https://blog.codinghorror.com/content/fig/uploads/2007/10/6a0120a85dcdae970b01287770273e970c-pi.png){alt='Left Join'}

A left join is performed in pandas by calling the same `merge` function used for
inner join, but using the `how='left'` argument:

```python
merged_left = pd.merge(left=places_df,right=authors_df, how='left', left_on='TCP', right_on='TCP')
merged_left

**OUTPUT:**
      TCP             Place                                             Author
0  A00002            London                         Aylett, Robert, 1583-1655?
1  A00005            London  Higden, Ranulf, d. 1364. Polycronicon. English...
2  A00007            London             Higden, Ranulf, d. 1364. Polycronicon.
3  A00008  The Netherlands?          Wood, William, fl. 1623, attributed name.
4  A00011         Amsterdam                                                NaN
```

The result DataFrame from a left join (`merged_left`) looks very much like the
result DataFrame from an inner join (`merged_inner`) in terms of the columns it
contains. However, unlike `merged_inner`, `merged_left` contains the **same
number of rows** as the original `place_sub` DataFrame. When we inspect
`merged_left`, we find there are rows where the information that should have
come from `authors_df` (i.e., `Author`) is missing (they contain NaN values):

```python
 merged_inner[ pd.isnull(merged_inner.Author) ]
**OUTPUT:**
        TCP Author      Place
4    A00011    NaN  Amsterdam
6    A00014    NaN     London
8    A00018    NaN   Germany?
```

These rows are the ones where the value of `Author` from `authors_df` does not occur in `places_df`.

### Other join types

The pandas `merge` function supports two other join types:

- Right (outer) join: Invoked by passing `how='right'` as an argument. Similar
  to a left join, except *all* rows from the `right` DataFrame are kept, while
  rows from the `left` DataFrame without matching join key(s) values are
  discarded.
- Full (outer) join: Invoked by passing `how='outer'` as an argument. This join
  type returns the all pairwise combinations of rows from both DataFrames; i.e.,
  the result DataFrame will `NaN` where data is missing in one of the dataframes.
  This join type is very rarely used.

## Final Challenges

:::::::::::::::::::::::::::::::::::::::  challenge

### Challenge - Distributions

Create a new DataFrame by joining the contents of the `authors.csv` and
`places.csv` tables. Calculate the:

1. Number of unique places
2. Number of books that do not have a known place
3. Number of books that do not have either a known place or author

:::::::::::::::  solution

### Solution to challenge

```python
merged = pd.merge(
                  left=pd.read_csv("authors.csv"),
                  right=pd.read_csv("places.csv"),
                  left_on="TCP",
                  right_on="TCP"
                  )
# Part 1: number of unique places - we can use the .nunique() method
num_unique_places = merged["Place"].nunique()
# Part 2: we can take advantage of the behaviour that the .count() method
#         excludes NaN values. So .count() gives us the number that have place
#         values
num_no_place = len(merged) - merged["Place"].count()
# Part 3: This needs us to check both columns and combine the resulting masks
#         Then  we can use the trick of converting boolean to int, and summing, 
#         to convert the combined mask to a number of True values
no_author = pd.isnull(merged["Author"]) # True where is null
no_place = pd.isnull(merged["Place"])
neither = no_author & no_place
num_neither = sum(neither)
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


