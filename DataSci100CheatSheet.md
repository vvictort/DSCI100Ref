# Cheat Sheet Midterm 1

>## Tidyverse (load, modify and plot data)

### <ins>**Loading the Data**</ins>
| Function           | Description                             | Usage                                     |
| :----------------- | :-------------------------------------- | :---------------------------------------- |
| `read_csv()`       | Read comma separated files              | `read_csv("data/test.csv")`               |
| `read_csv2()`      | Read semicolon separated files          | `read_csv2("data/test.csv")`              |
| `read_tsv()`       | Read tab separated files                | `read_tsv("data/test.tsv")`               |
| `read_delim()`     | Read any delimeters                     | `read_delim("data/test.csv", delim = "")` |
| `read_excel()`*    | Read Excel files                        | `read_excel("data/test.xlsx")`            |
| `nrow()`           | Count # of rows of data frame or vector | `nrow(data)` **or** `nrow(vector)`        |
| `download.file()`* | Download the file directly to local PC  | `download.file(url, "data/test.csv")`     |
**<ins>Note:</ins>** 
1.  `read_delim()` can have many arguments
    1. `delim` -> specify the delimeters
       - Must use _special_ (escaped) characters (i.e. `"\t"`)  
    2. `col_names` -> specify if you want R to give you a col_names
       - You can also create a vector via `c()` and make it your col_names
       -  Can be FALSE (indicating no headers; default is TRUE)
    3. `skip = 3` -> tell read_*() to skip # rows
       - Can be any number
    4. `file = path` -> specify the path of the a file
       - An ***absolute*** path start with `/` (i.e. `"/root/folder/test.csv"`)
         -  Starting from <ins>root</ins> folder      
       - A ***relative*** path start with a name (i.e. `"curr/fold/text.csv"`)
         - Starting from the <ins>current</ins> directory
2. `read_excel` ***needs*** `library(readxl)` (not `tidyverse`)
   1. `readxl` is a package that is **not** in `tidyverse`
   2. Others require package `readr` that is already exist in `tidyverse`
   3. Remember to specify `sheet` if needed
   
**<ins>Reading from Database</ins>**
1. SQLite Database
```R
# provide a channel that connect to the database
library(DBI)

canlang_conn <- dbConnect(RSQLite::SQLite(), "data/can_lang.db")

# show what Tables are in this Database
tables <- dbListTables(canlang_conn) 
tables

# make a reference to the table so that we can perform *some* operations on this table
library(dbplyr)

lang_db <- tbl(canlang_conn, "lang")
lang_db

# collect the database (turning it into R object)
collect(lang_db)
write_csv(lang_db, "data/lang_db.csv")
```
2. PostgreSQL Database
```R
# connect to RPostgres
library(RPostgres)
canmov_conn <- dbConnect(RPostgres::Postgres(), dbname = "can_mov_db",
                        host = "fakeserver.stat.ubc.ca", port = 5432,
                        user = "user0001", password = "abc123")
# see all the tables (there are 10 here)
dbListTables(canmov_conn)

# we pick "Ratings" table
ratings_db <- tbl(canmov_conn, "ratings")
ratings_db

# perform some dplyr operations
avg_rating_db <- select(ratings_db, average_rating)
avg_rating_db

# collect
avg_rating_data <- collect(avg_rating_db)
```
**Note:** PostgreSQL (also called Postgres) is a very popular and open-source option for relational database software. 
Unlike SQLite, PostgreSQL uses a client–server database engine, as it was designed to be used and accessed on a network. 
This means that you have to provide more information to R when connecting to Postgres databases.
   - This means that it can be accessed remotely since it exists on a network
   - Thereby, allowing multiple people to access it (i.e. enhance scalability) 

### <ins>**Modifying the Data**</ins> (Wrangling more so)
| Function      | Description                                                                 | Usage                                        |
| :------------ | :-------------------------------------------------------------------------- | :------------------------------------------- |
| `filter()`    | Filter the data based on conditional (**rows**)                             | `filter(data, cond == conditional)`          |
| `select()`    | Select the columns within the given data (**columns**, `start_with("...")`) | `select(data, colname1, colname2)`           |
| `mutate()`    | Create a **new column** containing the values caluclated                    | `mutate(data, new_column = old_column * 10)` |
| `arrange()`   | Ordering the columns (default is ascending)                                 | `arrange(data, by = desc(col1))`             |
| `slice()`     | Extra rows based on the range of #                                          | `slice(data, 1:10)`                          |
| `rename()`    | Rename the headers                                                          | `rename(data, new_name = old_name)`          |
| `as_tibble()` | Convert the data frame to tibble for `tidyverse`                            | `as_tibble(old_df)`                          |
| `group_by()`  | Group rows within the columns (do not change data)                          | `group_by(data, col_name)`                   |
| `summarize()` | Compute a summary statistics via **new** data frame                         | Look below `4`                               |
| `map()`       | Apply a function to **all** columns in a data frame                         | `map(max)`                                   |

There ways to modify the data in a more complex case:
1. **`pivot_wider`** &rarr; if rows represent more than one observation
```R
pivot_wider(data,
			names_from = old_column,
			values_from = old_cells)	
```
2. **`pivot_longer`** &rarr; if columns have same variable
```R
pivot_longer(data,
			 cols = col1:col3,
			 names_to = "new_name",
			 values_to = "new_values_name")
```
3. **`separate`** &rarr; if a cells have ***two*** values
```R
separate(data,
		 col = value,
		 into = c("new_col1", "new_col2"),
		 sep = "/", # can be any char
		 convert = TRUE) # default data-type is chr
```
4. **`summarize()`** 
```R
group_by(region_lang, region) |>
  summarize(
    min_most_at_home = min(most_at_home),
    max_most_at_home = max(most_at_home)
    )
```

### <ins>**Graphing**</ins>
**Things to remember for Visualization:**
   - Scatter Plots &rarr; between two quantitative variables
   - Line Plots &rarr; trends with respect to an independent and order quantity (i.e. time)
   - Bar Plots &rarr; comparing amounts
   - Histogram &rarr; distribution of **one** quantitative variable (i.e. all of its possible values and occurence)

**Rules of Thumbs for Visualization**
   1. Minimize Noise
      - Use colors sparingly
      - Try not to overplot
      - Only make the plot as you need
      - Try not to adjust the zoom if the difference is small, sometimes small difference means something 
   2. Convey the Message 
      - Use colors that are suitable for color-blinded individuals
      - Ensure graph properties (i.e. texts, symbols, axis, etc.) are big enough to read
      - Use legends and labels to make it easier for the reader to understand the graph
      - Make sure the visualization answer your questions
      - Redundancy can sometimes be helpful 

**Describing the Graph:** (key characteristics to include)
   1. Direction
      - Positive relationship (w/ x-axis)
      - Negative relationship (w/ x-axis)
      - Little to no relationship (w/ x-axis)  
   2. Strength
      - Strong (if data reliably shows a trend)
      - Weak (if shown otherwise)
   3. Shape 
      - Linear vs. Nonlinear (i.e. can you draw a straight line?) 
  
  **Graphing**
1. **Normal Scatter Plot graph**
```R
ggplot(data = the_data_name, aes(x = x_col, y = y_col)) + # Aesthetics
    geom_point() + # Geomtric
    xlab("The Name of X-axis") + # Labels
    ylab("The Name of Y-axis") +
    theme(text = element_text(size = 20)) # Themes
```
1. **Graph where data points are colored based on each class**
```R
ggplot(data = the_data_name, aes(x = x_col,  #Aesthetics
                                 y = y_col,
                                 color = class)) +
    geom_point(alpha = 0.6) +  # Geometric
    labs(x = "X-axis",  # Labels
         y = "Y-axis",
         color = "The name of legends",
         title = "Title of Graph") +
    theme(text = element_text(size = 20)) # Theme
```
1. **Graph with Color Brewer** -> Scatter Plot (ggplot2??)
```R
ggplot(data = the_data_name, aes(x = x_col,  #Aesthetics
                                 y = y_col,
                                 color = class)) +
    geom_point(alpha = 0.6) +  # Geometric
    labs(x = "X-axis",  # Labels
         y = "Y-axis",
         color = "The name of legends") +
    scale_fill_brewer(palette = "Set2") +
    theme(text = element_text(size = 20)) # Theme
```
1. **`library(lubridate)`** &rarr; for using dates
```R
library(lubridate) # a part of tidyverse, but not loaded in so load in now

ggplot(data, aes(x = col1, y = col2)) +
   geom_point() +
   labs(x = "Col1", y "Col2") +
   xlim(c("1990-02-01", "2017-01-02")) # must be a vector for xlim()
```
#### Geometric That You Can Replace
| Function                               | Description                                          | Note                      |
| :------------------------------------- | :--------------------------------------------------- | :------------------------ |
| `geom_point()`                         | Create Scatter Plot                                  |
| `geom_bar()`                           | Create bar graph                                     |                           |
| `geom_histogram`                       | Create histogram                                     |                           |
| `geom_line()`                          | Line graph                                           |                           |
| `geom_vline() `                        | Insert vertical line                                 |                           |
| `alpha = 0.6`                          | Transparency of points                               | an argument for **plot**  |
| `size = 2`                             | Change size of scatter plot                          | an argument               |
| `stat = identity`                      | Make heights of bar equals to amount ***y***         | an argument for **bar**   |
| `xlim(c(lower, upper))`                | Narrow the graph down to lower and upper bound       | a **LAYER**               |
| `scale_x/y_log10(labels = as_comma())` | Log10 the x-axis or y-axis                           | a **LAYER**               |
| `facet_grid(rows/columns = vars(Var))` | Make the plots in rows or columns                    | a **LAYER**               |
| `legend.position = "top"`              | Changing the position of legends around the plots    | an argument for **theme** |
| `legend.direction = "vertical"`        | Specify how the legends are stacked (vert. or hori.) | an argument for **theme** |

- Special Notes to Consider (when making a graph):
  1. You can change the x-axis and y-axis to make a bar graph more clean
  2. Always use color palette that suitable for color-blind (look at [3])
#### Repr (format the plot?)
 ```R
 options(repr.plot.width = 8, repr.plot.height = 8)
 ```
>## tidymodels (Modelling)
 # Todo
 - [ ] SQLite (loading, reading and modifying) -> end of Chapter 2
 - [ ] Rowise, mutate, across, group_by (end of Chapter 3)
 - [ ] How to interpret (Chapter 4)
 - [ ] fill = fill

 
