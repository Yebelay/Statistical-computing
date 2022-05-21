## Introduction to R-Part 3
## dplyr base R


#### Compares dplyr functions to their base R equivalents.

- This helps those familiar with base R understand better what dplyr does, and / or

- shows dplyr users how you might express the same ideas in base R code. 

1.  In dplyr you don't need to use `$` to refer to columns in the "current" data frame. 
    This behaviour is inspired by the base functions `subset()` and `transform()`.
    
1.  dplyr solutions tend to use a variety of single purpose verbs, while base 
    R solutions typically tend to use `[` in a variety of ways, depending on the 
    task at hand. 
    
1.  Multiple dplyr verbs are often strung together into a pipeline by `%>%`.
    In base R, you'll typically save intermediate results to a variable that
    you either discard, or repeatedly overwrite.

---

# Compare dplyr vs base 

- For one table verbs

.pull-left[
**<span style="color:red">dplyr</span>**
1. **<span style="color:blueviolet">arrange(df, x)</span>**
1. **<span style="color:blueviolet">distinct(df, x)</span>**
1. **<span style="color:blueviolet">filter(df, x)</span>**
1. **<span style="color:blueviolet">mutate(df, z = x + y)</span>** 
1. **<span style="color:blueviolet">pull(df, 1)</span>**
1. **<span style="color:blueviolet">pull(df, x)</span>**
1. **<span style="color:blueviolet">rename(df, y = x)</span>** 
1. **<span style="color:blueviolet">relocate(df, y)</span>**
1. **<span style="color:blueviolet">select(df, x, y)</span>**
1. **<span style="color:blueviolet">select(df, starts_with("x"))</span>**
1. **<span style="color:blueviolet">summarise(df, mean(x))</span>**
1. **<span style="color:blueviolet">slice(df, c(1, 2, 5))</span>**
]
.pull-rigt[
**<span style="color:red">base</span>** 
1. **<span style="color:blue">df[order(x), , drop = FALSE]</span> **
1. **<span style="color:blue">df[!duplicated(x),,drop=FALSE]`,`unique()</span>**
1. **<span style="color:blue">df[which(x), , drop = FALSE]`, `subset()</span>**
1. **<span style="color:blue">df$z <- df$x + df$y`, `transform()</span>**
1. **<span style="color:blue">df[[1]]</span>**
1. **<span style="color:blue">df$x</span> **
1. **<span style="color:blue">names(df)[names(df)== "x"]<- "y"</span>**
1. **<span style="color:blue">df[union("y", names(df))]</span>**
1. **<span style="color:blue">df[c("x", "y")], subset()</span>**               
1. **<span style="color:blue">df[grepl(names(df), "^x")]</span>**
1. **<span style="color:blue">mean(df$x)`, `tapply()`, `aggregate()`, `by()</span>**
1. **<span style="color:blue">df[c(1, 2, 5), , drop = FALSE]</span>**

]

---

# `arrange()`: 

- Arrange rows by variables

`dplyr::arrange()` orders the rows of a data frame by the values of one or more columns: 

```{r , message = FALSE, eval=FALSE}
library(dplyr)
mtcars <- as_tibble(mtcars)
iris <- as_tibble(iris)
```

```{r , message = FALSE, eval=FALSE}
mtcars %>% arrange(cyl, disp)
```

The `desc()` helper allows you to order selected variables in descending order:

```{r , message = FALSE, eval=FALSE}
mtcars %>% arrange(desc(cyl), desc(disp))#<<
```

We can replicate in base R by using `[` with `order()`:

```{r , message = FALSE, eval=FALSE}
mtcars[order(mtcars$cyl, mtcars$disp), , drop = FALSE]#<<
```

---

- Note the use of `drop = FALSE` is to get dataframe output in a data frame with a single column.

- Base R does not provide a convenient and general way to sort individual variables in descending order, so you have two options:

* For numeric variables, you can use `-x`.
* You can request `order()` to sort all variables in descending order.

```{r, results = FALSE}
mtcars[order(mtcars$cyl, mtcars$disp, decreasing = TRUE), , drop = FALSE] #<<
mtcars[order(-mtcars$cyl, -mtcars$disp), , drop = FALSE] #<<
```

---

# `distinct()`:

- Select distinct/unique rows

`dplyr::distinct()` selects unique rows:

```{r , message = FALSE, eval=FALSE}
df <- tibble(
  x = sample(10, 100, rep = TRUE),
  y = sample(10, 100, rep = TRUE))
df %>% distinct(x) # selected columns
df %>% distinct(x, .keep_all = TRUE) # whole data frame
```

- There are two equivalents in base R, depending on whether you want the whole data frame, or just selected variables:

```{r , message = FALSE, eval=FALSE}
unique(df["x"]) # selected columns
df[!duplicated(df$x), , drop = FALSE] # whole data frame
```

---

# `filter()`: 

- Return rows with matching conditions

- `dplyr::filter()` selects rows where an expression is `TRUE`:

```{r , message = FALSE, eval=FALSE}
starwars %>% filter(species == "Human")
starwars %>% filter(mass > 1000)
starwars %>% filter(hair_color == "none" & eye_color == "black")#<<
```

- The closest base equivalent (and the inspiration for `filter()`) is `subset()`:

```{r , message = FALSE, eval=FALSE}
subset(starwars, species == "Human")#<<
subset(starwars, mass > 1000)#<<
subset(starwars, hair_color == "none" & eye_color == "black")#<<
```

- You can also use `[` but this also requires the use of `which()` to remove `NA`s:

```{r , message = FALSE, eval=FALSE}
starwars[which(starwars$species == "Human"), , drop = FALSE]#<<
```

---

# `mutate()`: 

- Create or transform variables

- `dplyr::mutate()` creates new variables from existing variables:

```{r , message = FALSE, eval=FALSE}
df %>% mutate(z = x + y, z2 = z ^ 2)
```

- The closest base equivalent is `transform()`, but note that it cannot use freshly created variables:

```{r , message = FALSE, eval=FALSE}
head(transform(df, z = x + y, z2 = (x + y) ^ 2))
```

Alternatively, you can use `$<-`:

```{r , message = FALSE, eval=FALSE}
mtcars$cyl2 <- mtcars$cyl * 2#<<
mtcars$cyl4 <- mtcars$cyl2 * 2
```

---

When applied to a grouped data frame, `dplyr::mutate()` computes new variable once per group:

```{r , message = FALSE, eval=FALSE}
gf <- tibble(g = c(1, 1, 2, 2), x = c(0.5, 1.5, 2.5, 3.5))
gf %>% group_by(g) %>% 
  mutate(x_mean = mean(x), x_rank = rank(x))#<<
```

To replicate this in base R, you can use `ave()`:

```{r , message = FALSE, eval=FALSE}
transform(gf, #<<
  x_mean = ave(x, g, FUN = mean), 
  x_rank = ave(x, g, FUN = rank))
```

---

# `pull()`: 

- Pull out a single variable

- `dplyr::pull()` extracts a variable either by name or position:

```{r , message = FALSE, eval=FALSE}
mtcars %>% pull(1)
mtcars %>% pull(cyl)
```

- This equivalent to `[[` for positions and `$` for names:

```{r , message = FALSE, eval=FALSE}
mtcars[[1]]
mtcars$cyl
```

---

# `relocate()`: 

- Change column order

- `dplyr::relocate()` makes it easy to move a set of columns to a new position (by default, the front):

```{r , message = FALSE, eval=FALSE}
# to front
mtcars %>% relocate(gear, carb) 
# to back
mtcars %>% relocate(mpg, cyl, .after = last_col()) 
```

- We can replicate this in base R with a little set manipulation:

```{r , message = FALSE, eval=FALSE}
mtcars[union(c("gear", "carb"), names(mtcars))]
to_back <- c("mpg", "cyl")
mtcars[c(setdiff(names(mtcars), to_back), to_back)]
```

Moving columns to somewhere in the middle requires a little more set twiddling.

---

# `rename()`: 

- Rename variables by name

- `dplyr::rename()` allows you to rename variables by name or position:

```{r , message = FALSE, eval=FALSE}
iris %>% rename(sepal_length = Sepal.Length, sepal_width = 2)
```

- Renaming variables by position is straight forward in base R:

```{r , message = FALSE, eval=FALSE}
iris2 <- iris
names(iris2)[2] <- "sepal_width"
```

- Renaming variables by name requires a bit more work:

```{r , message = FALSE, eval=FALSE}
names(iris2)[names(iris2) == "Sepal.Length"] <- "sepal_length"
```

---

# `rename_with()`: 

- Rename variables with a function

- `dplyr::rename_with()` transform column names with a function:

```{r , message = FALSE, eval=FALSE}
iris %>% rename_with(toupper)
```

- A similar effect can be achieved with `setNames()` in base R:

```{r , message = FALSE, eval=FALSE}
setNames(iris, toupper(names(iris)))
```

---

# `select()`: 

- Select variables by name

- `dplyr::select()` subsets columns by position, name, function of name, or other property:

```{r , message = FALSE, eval=FALSE}
iris %>% select(1:3)
iris %>% select(Species, Sepal.Length)
iris %>% select(starts_with("Petal"))#<<
iris %>% select(where(is.factor))#<<
```

- Subsetting variables by position is straightforward in base R:

```{r , message = FALSE, eval=FALSE}
iris[1:3] # single argument selects columns; never drops
iris[1:3, , drop = FALSE]
```

- You have two options to subset by name:

```{r , message = FALSE, eval=FALSE}
iris[c("Species", "Sepal.Length")]
subset(iris, select = c(Species, Sepal.Length))
```

---

Subsetting by function of name requires a bit of work with `grep()`:

```{r , message = FALSE, eval=FALSE}
iris[grep("^Petal", names(iris))]
```

And you can use `Filter()` to subset by type:

```{r , message = FALSE, eval=FALSE}
Filter(is.factor, iris)
```

---

# `summarise()`: 

- Reduce multiple values down to a single value

- `dplyr::summarise()` computes one or more summaries for each group:

```{r , message = FALSE, eval=FALSE}
mtcars %>% 
  group_by(cyl) %>% 
  summarise(mean = mean(disp), n = n())#<<
```

- I think the closest base R equivalent uses `by()`. 
- Unfortunately `by()` returns a list of data frames, but you can combine them back together again with `do.call()` and `rbind()`:

```{r , message = FALSE, eval=FALSE}
mtcars_by <- by(mtcars, mtcars$cyl, function(df) {
  with(df, data.frame(cyl = cyl[[1]], mean = mean(disp), n = nrow(df)))
})
do.call(rbind, mtcars_by)#<<
```

---

- `aggregate()` comes very close to providing an elegant answer:

```{r , message = FALSE, eval=FALSE}
agg <- aggregate(disp ~ cyl, mtcars, function(x) c(mean = mean(x), n = length(x)))
```

- But unfortunately while it looks like there are `disp.mean` and `disp.n` columns, it's actually a single matrix column:

```{r , message = FALSE, eval=FALSE}
str(agg)
```

You can see a variety of other options at <https://gist.github.com/hadley/c430501804349d382ce90754936ab8ec>.

---

# `slice()`: 

- Choose rows by position

- `slice()` selects rows with their location:

```{r , message = FALSE, eval=FALSE}
slice(mtcars, 25:n())
```

- This is straightforward to replicate with `[`:

```{r , message = FALSE, eval=FALSE}
mtcars[25:nrow(mtcars), , drop = FALSE]
```

---

# Merging datasets

- When we want to merge two data frames, `x` and `y`), we have a variety of different ways to bring them together. 

- Various base R `merge()` calls are replaced by a variety of dplyr `join()` functions.

.pull-left[
**<span style="color:red">dplyr</span>**
1. **<span style="color:blueviolet">inner_join(df1, df2)</span>** 
1. **<span style="color:blueviolet">left_join(df1, df2) </span>**
1. **<span style="color:blueviolet">right_join(df1, df2)</span>** 
1. **<span style="color:blueviolet">full_join(df1, df2)</span>** 
1. **<span style="color:blueviolet">semi_join(df1, df2)</span>**  
1. **<span style="color:blueviolet">anti_join(df1, df2)</span>**
]
.pull-left[
**<span style="color:red">base</span>**
1. **<span style="color:blueblueviolet">merge(df1, df2)</span>**
1. **<span style="color:blueblueviolet">merge(df1, df2, all.x = TRUE)</span>**
1. **<span style="color:blueblueviolet">merge(df1, df2, all.y = TRUE)</span>**
1. **<span style="color:blueblueviolet">merge(df1, df2, all = TRUE)</span>**
1. **<span style="color:blueblueviolet">df1[df1$x %in% df2$x, ,drop=FALSE]</span>**
1. **<span style="color:blueblueviolet">df1[!df1$x %in% df2$x, ,drop=FALSE]</span>** 
]

---

# Mutating joins

- dplyr's `inner_join()`, `left_join()`, `right_join()`, and `full_join()` add new columns from `y` to `x`, matching rows based on a set of "keys", and differ only in how missing matches are handled. 

- They are equivalent to calls to `merge()` with various settings of the `all`, `all.x`, and `all.y` arguments. 

- The main difference is the order of the rows:

* dplyr preserves the order of the `x` data frame.
* `merge()` sorts the key columns.

---

# Filtering joins

- dplyr's `semi_join()` and `anti_join()` affect only the rows, not the columns:

```{r , message = FALSE, eval=FALSE}
band_members %>% semi_join(band_instruments)
band_members %>% anti_join(band_instruments)
```

- They can be replicated in base R with `[` and `%in%`:

```{r , message = FALSE, eval=FALSE}
band_members[band_members$name %in% band_instruments$name, , drop = FALSE]
band_members[!band_members$name %in% band_instruments$name, , drop = FALSE]
```

- Semi and anti joins with multiple key variables are considerably more challenging to implement.