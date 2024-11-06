*# Data Science Review Sheet

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
    1. `delim` &rarr; specify the delimeters
       - Must use _special_ (escaped) characters (i.e. `"\t"`)  
    2. `col_names` &rarr; specify if you want R to give you a col_names
       - You can also create a vector via `c()` and make it your col_names
       -  Can be FALSE (indicating no headers; default is TRUE)
    3. `skip = 3` &rarr; tell read_*() to skip # rows
       - Can be any number
    4. `file = path` &rarr; specify the path of the a file
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
```r
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
```r
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
| `mutate()`    | Create a **new column** containing the values calculated                    | `mutate(data, new_column = old_column * 10)` |
| `arrange()`   | Ordering the columns (default is ascending)                                 | `arrange(data, by = desc(col1))`             |
| `slice()`     | Extra rows based on the range of #                                          | `slice(data, 1:10)`                          |
| `rename()`    | Rename the headers                                                          | `rename(data, new_name = old_name)`          |
| `as_tibble()` | Convert the data frame to tibble for `tidyverse`                            | `as_tibble(old_df)`                          |
| `group_by()`  | Group rows within the columns (do not change data)                          | `group_by(data, col_name)`                   |
| `summarize()` | Compute a summary statistics via **new** data frame                         | Look below `4`                               |
| `map()`       | Apply a function to **all** columns in a data frame                         | `map(max)`                                   |

There ways to modify the data in a more complex case:
1. **`pivot_wider`** &rarr; if rows represent more than one observation
```r
pivot_wider(data,
		 names_from = old_column,
		 values_from = old_cells)	
```
2. **`pivot_longer`** &rarr; if columns have same variable
```r
pivot_longer(data,
		 cols = col1:col3,
		 names_to = "new_name",
		 values_to = "new_values_name")
```
3. **`separate`** &rarr; if a cells have ***two*** values
```r
separate(data,
	 col = value,
	 into = c("new_col1", "new_col2"),
	 sep = "/", # can be any char
	 convert = TRUE) # default data-type is chr (convert to right data type)
```
4. **`summarize()`** 
```r
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
      - Avoid overplotting
      - Only make the plot as you need
      - Avoid adjusting the zoom if the difference is small, as small differences can be meaningful
   2. Convey the Message 
      - Use colors that are suitable for color-blind individuals
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
```r
ggplot(data = the_data_name, aes(x = x_col, y = y_col)) + # Aesthetics
    geom_point() + # Geometric
    xlab("The Name of X-axis") + # Labels
    ylab("The Name of Y-axis") +
    theme(text = element_text(size = 20)) # Themes
```
1. **Graph where data points are colored based on each class**
```r
ggplot(data = the_data_name, aes(x = x_col,  # Aesthetics
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
```r
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
```r
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
 ```r
 options(repr.plot.width = 8, repr.plot.height = 8)
 ```
>## Tidymodels (Modelling I and II)
**Rule of Thumbs**:
1. Only use your **training set** to evaluate your classier (nothing else)

**Evaluate Your Model/Classifier**:\
There are many different ways to *evaluate* your classifier. In this course,
you will only use `accuracy`, `precision` and `recall`.

**Retuning Your Model**\
Tuning is important for your classifier and model.

**Ways to Train K-NN Model**
1. `workflow()` (Classification I)
```r
# load the unscaled cancer data
# and make sure the response variable, Class, is a factor
unscaled_cancer <- read_csv("data/wdbc_unscaled.csv") |>
  mutate(Class = as_factor(Class)) |>
  mutate(Class = fct_recode(Class, "Malignant" = "M", "Benign" = "B"))

# create the K-NN model
knn_spec <- nearest_neighbor(weight_func = "rectangular", neighbors = 7) |>
  set_engine("kknn") |>
  set_mode("classification")

# create the centering / scaling recipe
uc_recipe <- recipe(Class ~ Area + Smoothness, data = unscaled_cancer) |>
  step_scale(all_predictors()) |>
  step_center(all_predictors())
```
>## Tidymodels (Regression I + II)
What happens if you want to predict a <ins>quantitative</ins> value instead of a label?
   1. *K-NN Regression* 
   2. *K-NN Linear Regression*.

How can we determine (1) the K-value and (2) if our model is any "good"?
   - Same as K-NN *Classification*
      1. Training Set vs. Testing Set (K-value)
      2. Accuracy &rarr; Cross-Validation + (lowest) RMSPE (effectiveness of model)

**Root Mean Squared Prediction Error (RMSPE)**\
Essentially, this is the "error" between the predicted value and the observed value for 
all observations. By error, this refers to the distance between the predicted and observed values. 
We will take the lowest like 'accuracy' in K-NN Classification.
   $$ 
   \sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y_i})^2}
   $$
   - $n$ = # of observations
   - $y_i$ = the observed value at $i^{th}$ observation
   - $\hat{y_i}$ = the predicted value at $i^{th}$ observation

<ins>**Note**</ins>: 
   1. The ***units*** matches with the ***predicted*** variables, and not out of 1 (i.e. accuracy in Classification).
   2. These errors are the ***vertical*** distances, since $y$-variable is our *predicted*
   3. RMSE and RMSPE are two <ins>**different**</ins> things (but same formula) 
      - RMSE &rarr; how well the model fit our [training] data
      - RMSPE &rarr; how well the model *generalizes* <ins>unseen</ins> data

**K-NN Regression**
- <ins>Disadvantage</ins>:
   - Can be slow if data is large
   - May not perform well if there are *too many predictors*
   - Unable to *extrapolate* (i.e. predicting values *beyond* the data set)
   - The trendline from the data set is hard to interpret (i.e. wringly lines)
- <ins>Advantage</ins>: (same as K-NN Classification)
  - Few assumptions are needed based on the shape of the graph
  - Simple, intuitive algorithm,
  - Requires few assumptions about what the data must look like
  - Works well with non-linear relationships (i.e., if the relationship is not a straight line).

```r
# Split the data
sacramento_split <- initial_split(sacramento, prop = 0.75, strata = price)
sacramento_train <- training(sacramento_split)
sacramento_test <- testing(sacramento_split)

# Cross-Validation and lowest RMSPE
sacr_recipe <- recipe(price ~ sqft, data = sacramento_train) |>
  step_scale(all_predictors()) |>
  step_center(all_predictors())

sacr_spec <- nearest_neighbor(weight_func = "rectangular",
                              neighbors = tune()) |>
  set_engine("kknn") |>
  set_mode("regression")

sacr_vfold <- vfold_cv(sacramento_train, v = 5, strata = price)

sacr_wkflw <- workflow() |>
  add_recipe(sacr_recipe) |>
  add_model(sacr_spec)

sacr_wkflw

gridvals <- tibble(neighbors = seq(from = 1, to = 200, by = 3))

sacr_results <- sacr_wkflw |>
  tune_grid(resamples = sacr_vfold, grid = gridvals) |>
  collect_metrics() |>
  filter(.metric == "rmse")

# show the results
sacr_results

# show only the row of minimum RMSPE
sacr_min <- sacr_results |>
  filter(mean == min(mean))

sacr_min # K = 52

# Train the model on ENTIRE training data set
kmin <- sacr_min |> pull(neighbors)

sacr_spec <- nearest_neighbor(weight_func = "rectangular", neighbors = kmin) |>
  set_engine("kknn") |>
  set_mode("regression")

sacr_fit <- workflow() |>
  add_recipe(sacr_recipe) |>
  add_model(sacr_spec) |>
  fit(data = sacramento_train)

# Evalution
sacr_summary <- sacr_fit |>
  predict(sacramento_test) |>
  bind_cols(sacramento_test) |>
  metrics(truth = price, estimate = .pred) |>
  filter(.metric == 'rmse')

sacr_summary
```

**K-NN Linear Regression**
- <ins>Disadvantage</ins>:
  - Can be slow if data is large (**same**)
  - May not perform well if there are *too many predictors* (**same**)
- <ins>Advantage</ins>:
  - Is able to *extrapolate* (as you you have an equation to work with)
  - More interpretable overall trendline
  
***Insert Code Here***

 # Todo
 - [x] SQLite (loading, reading and modifying) -> end of Chapter 2
 - [ ] Rowise, mutate, across, group_by (end of Chapter 3)
 - [x] How to interpret (Chapter 4)
 - [ ] fill = fill

 
*