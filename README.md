# Pandas CheatSheet

Handy tips and tricks for working with Pandas DataFrames. Focused around the basics of exploring a dataset. WIP, may expand later.

## Notebooks

There's a Jupyter Notebook included with this repo, both for quick reference and for playing around with example data.

 - [Pandas DataFrames](<Pandas DataFrames.ipynb>) - some basic examples of key commands with dummy datasets
 - [Worked Example](<Worked Example.ipynb>) - some experimentation in a real dataset (WIP, to be expanded)


## Useful Links

 - [Pandas API Docs](https://pandas.pydata.org/pandas-docs/stable/reference/index.html)


## Getting started

This project assumes you have Python 3 on your system, and has been tested on OS X, though *should* work on Linux as well. Other pre-requisites:

1. `pip` - the [python package manager](https://pip.pypa.io/en/stable/). Should come with official Python 3 >= 3.4.
2. [`virtualenv`](https://virtualenv.pypa.io/en/stable/) - A tool for creating sandboxes for python packages. Can be installed with `pip install virtualenv`


### Set up a Virtual Environment

Create a virtual environment for the project. Feel free to use whatever directory you want, I tend to use `bin/env` within the project directory.

`> virtualenv bin/env`

Next, activate it. The `activate` script lives in the `bin` dir of the virtual environment directory, and can be run either with the `source` command, or the shorthand `.`:

`> source bin/env/bin/activate`

`> . bin/env/bin/activate`


### Install Dependencies

The dependencies for this project are basic, and can be found in `requirements.txt`. Install them with pip (making sure your virtualenv is active):

`(env)> pip install -r requirements.txt`


## Using Pandas

N.B. `pandas` is normally aliased as `pd` for brevity, so most Pandas code will import the library as so:

`import pandas as pd`


### Handy options

It can be useful to limit the max number of rows/columns to display:

```
pd.set_option("display.max_row", 1000)
pd.set_option("display.max_columns", 50)
```

If plotting graphs in a notebook environment, it's useful to make them plot inline:

`%matplotlib inline`


### I/O

Reading from CSV to a dataframe:

`df = pd.read_csv(<filehandle>)`

`df = pd.read_csv(<path to file>)`

This will read in a CSV and try to infer datatypes.

*Useful params:*

 - `sep` (default=`','`) - use a custom separator, multi-character separators will use the regex engine
 - `header` - Use particular rows as column names. Use `0` to replace header names with your own values
 - `names` - provide replacement header names (as an array)
 - `skiprows` - skip <n> rows at the start of the file, or particular rows if given a list. Can also provide a function
 - `skip_blank_lines` (default=`True`) - If `True`, skips blank lines (otherwise read as NaNs)


Create a DataFrame from lists:

```
a = [["a", "b", "c"], [1, 2, 3], [4, 5, 6]]
df = pd.DataFrame(a)
```

This will just create a DF with no column headers from the provided data.

*Useful params:*
 - `dtype` - tell Pandas what type the columns are, or force them to a particular type.


### Basic DataFrame Operations

A quick overview of a DataFrame:

`df.describe()`

Returns counts, mean, standard deviation, min, max, and 25/50/75% percentiles for each column.

Look at the first/last few rows:

`df.head()`

`df.tail()`

Listing columns in a DataFrame:

`df.columns`

Checking the types of each column:

`df.dtypes`

Column access:

`df["column"]`

OR

`df.column`

Multi-column selection:

`df[["column1", "column2", "column3"]]`

Checking the size of a DataFrame:

`df.shape`

Finding how many unique values there are in a particular column:

`len(df.<column>.unique())`

Drop a column (to simplify data, or for removing columns with little/no information):

`df.drop(<column>)`

*Useful params:*
 - `inplace` (default `False`) - whether to perform the operation on the same DF, or return it as a new DF.
 - `columns` - single label or list of labels. Shorthand for `(labels, axis=1)`.
 - `index` - single label or list of labels. Shorthand for `(labels, axis=0)`.

Define an existing column as the index of the DataFrame (can be useful for lookups, etc).

`df.set_index(<column>)`

*Useful params:*
 - `inplace` (default `False`) - whether to perform the operation on the same DF, or return it as a new DF.

It can be useful to check uniqueness before doing this. DF indices don't have to be unique, but operations will generally perform better if they are.

```
> df[<column>].is_unique
True
```

Renaming columns is also fairly common:

`df.rename(columns=<new_names>, inplace=True)`

Converting columns (sometimes crudely) into numeric values from a collection of strings is a pretty common cleaning operation.

`df[<new_col>] = pd.to_numeric(df[<col_to_parse>].str.extract(r'<some regex>', expand=False))`

(`expand` here will return a DF with a column per regex group if set to `True`, otherwise a Series if there's only one, and a DF if there are multiple)

Numpy `where` is also very powerful for processing strings, with a simple `condition`, `then`, `else` syntax:

```
col_to_parse = df[<some_col>]
search = col_to_parse.str.contains["search"]
df[<new_col>] = np.where(search, "replacement", col_to_parse.str.replace("-", " "))
```

So this searches `some_col` for `"search"`, replacing it with `"replacement"` if found. If it doesn't find `"search"`, it defaults to the other replace expression, which replaces hyphens with spaces.

`np.where` can also be chained:

`np.where(<check1>, <replacement1>, np.where(<check2>, <replacement2>, <default case>))`


#### Casting columns

Four main methods on DataFrames for casting:

 - `pd.to_numeric()` - converts columns to a numeric type. Has various options for ignoring errors, coercing, picking smallest datatype.
 - `df.astype()` - force types. Can do multi-type casting (`df.astype({"a": int, "b": complex})`). A rather blunt tool.
 - `df.infer_objects()` - infers types for columns of type `object`.
 - `pd.convert_dtypes()` - convert types to the best types that support missing values.

Example usage:

```
df["a"] = pd.to_numeric(df["a"])
```


#### Plotting data

There is a basic `df.plot()` function, but it's largely not useful as is.

To plot a bar chart of the counts of each value for a given column (for figuring out distribution of data):

`df.groupby(<column>).count().plot(kind="bar")`

Or a histogram of the entire DataFrame:

`df.hist()`

*Useful params:*
 - `bins` (default `10`) - The number of buckets/bins to group data into.

To plot all columns on the same chart:

`df.plot.hist()`

*Useful params:*
 - `bins` (default `10`) - The number of buckets/bins to group data into.
 - `alpha` (`float`, default `1.0`) - The transparency. Setting to < 1.0 makes overlapping visible.

Scatter plots are also a useful quick gauge of correlations/trends between columns. Again, this is built into Pandas:

`df.plot.scatter(x=<x-column>, y=<y-column>)`


#### Correlation

Pandas has column correlation built in, useful for identifying which columns may be related. Correlation is in a range from -1.0 to 1.0. 1.0 equals perfect positive correlation (when `x` goes up, `y` goes up), -1.0 is perfect negative correlation (when `x` goes up, `y` goes down), and values close to zero indicate weak/no correlation.

`df.corr()`

Returns a basic correlation method across all columns.

*Useful params:*
 - `method` (default `pearson`) - Which type of correlation to use [`pearson`, `kendall`, `spearman`]. Can also take a custom implementation.
 - `min_periods` (default `1`) - minimum number of observations to have a valid result. Only works with `pearson` and `spearman`.

Plotting correlations is usually good for eyeballing the trends. `seaborn` does this really well:

```
import seaborn as sn
import matplotlib.pyplot as plt

sn.heatmap(df.corr())
plt.show()
```


## References

I used a bunch of different DataScience blogs and articles to throw this together. Here are the main ones:

Histograms: 
 - https://data36.com/plot-histogram-python-pandas/

Correlation
 - https://re-thought.com/exploring-correlation-in-python/
 - https://datatofish.com/correlation-matrix-pandas/
 - https://towardsdatascience.com/correlation-is-simple-with-seaborn-and-pandas-28c28e92701e

Scatter plots:
 - https://data36.com/scatter-plot-pandas-matplotlib/

General column operations and common cleaning operations:
 - https://realpython.com/python-data-cleaning-numpy-pandas/