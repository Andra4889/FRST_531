---
title: "Exercise 3: Factor Analysis to Find Underlying Factors"
output:
    html_document: default
    word_document: default
    pdf_document: default 
---

**Course Instructor**: Dr. Suborna Ahmed

**Student Name**:

**Total Points**: 10

**Due**: check Canvas page for due date

**Submission**:

1. Write your code and answers in this RMD file
2. Knit your completed RMD file to WORD document
3. Check and upload your WORD document to Exercise 2 on Canvas

---------------------------------------------------------------

### Objectives

1. To become familiar with Factor Analysis as a tool used to uncover underlying
relationships in a set of x-variables.

2. To learn how to use different rotations as a means of achieving the simple
structure needed in order to be able to interpret the factors (and the factor
scores) and assign meaningful names to each of them.

### General Description

Information was gathered by Tracy Hooper on bird abundancies, along with
vegetation and other variables in a grassland area (a subset of her data is used in
FRST 531 with her approval). Using factor analysis, the underlying relationships
among __11 of the vegetation variables__ was analyzed. The 11 variables are:

* `mayvgcov` and `mayvght` (% cover and height of vegetation in May) 
* `junvgcov and `junvght` (% cover and height of vegetation in June); 

* and horizontal cover variables, specifically: `baregrnd`, `bryopht`, `forbs`, `grass`, `rocks`, `shrubs`, and `trees` 
all as __percents__.

The overall objective is to use factor analysis to 
**obtain a few indices to represent underlying factors, and to be able to give these underlying factors names**. 
These factors could give insights into the data (e.g., which plots gave high scores on Factor 1 versus low scores, and what does this mean ecologically?). Also, these factors
could be used in a subsequent analysis to replace these 11 variables.

However, for the factors to have meaning and to be able to name the factors, a
simple structure is needed. For example, if Factor 1 has high positive correlations
with `mayvgcov` and `junvgcov` perhaps this factor is “summer vegetation cover”.
However, if Factor 1 also has a moderately high positive correlation with `rocks`,
what does this mean? Instead another rotation may provide a more interpretable
and simple structure.

For this, you will first do the PCA. Then, you will use factor analysis first with no
rotation. Then, you will use orthogonal and oblique rotations to get a simple
solution. Remember that oblique rotations are much harder to interpret since the
factors are correlated (unless the correlation is very low?). Our preference is to
find a simple structure using orthogonal rotations if possible.

### Process

1. For this analysis, the data have been provided as `TRABREV2.txt` along with R
codes and questions as in this Rmd file.

2. You will need to run the R code and get the outputs from running the code.
There are two things you will need to change:

+ Change the working directory to the location of the data on
your computer, if you have not done so already.

+ Add in code for the 4th factor analysis run (object will be called `veg.fac4`)
by cutting and pasting code for the other factor analysis runs and then
modifying this for an equamax rotation. There are three other factor
analysis runs before this one (i.e., `veg.fac1` to `veg.fac3`) and one after
(i.e., `veg.fac5`) that are already in the code and there is a location
where to put the 4th run for `veg.fac4`.

3. The outputs will include:
+ The PCA on the 11 variables;
+ The unrotated factor solution, using two factors (`veg.fac1`)
+ Several rotated factor solutions (`veg.fac2`, `veg.fac3`, `veg.fac4` (you will
put the code in), and `veg.fac5` (the only oblique rotation).

4. You will be answering a **8 questions** to guide you through the outputs
and interpret the results. The questions are marked with **TODO** to help you quickly 
locate them using the search tool, since this Rmd file is quite long. As you answer 
the questions, paste in the relevant R outputs (and/or graphs) to support your answers.

----------------------------------------------------------------------------------
### R CODES

#### Introduction 
Data from an outside text file from birds in grasslands study by Tracy Hooper 
(subset of her data used by permission) is brought in to R from an outside 
space-delimited text file. Alternatively, you could bring the text file into 
EXCEL and save it as a `.csv` file. Using only a subset of the variables, the goal 
is to get a simple structure on a few factors in order to label and interpret these factors.
These factors would then commonly be passed to another analysis. We would prefer to
obtain a simple structure using orthogonal rotations, but sometimes this is not
possible. The R code starts with PCA first to indicate the contribution of
each principal component. Factor analysis (FA) then starts with two
factors and no rotation, which is like the PCA solution as presented in the
Factor analysis notes. Then, further rotations are used to try to get the
simple structure, starting with orthogonal and then oblique rotations.  

#### PART 1: Clean out all R objects before starting. 
Then set the directory for the data (and any files saved) and read the data. 
We assign it to an R object called `vegdat`. Do a few checks on the data. 

```{r}
rm(list = ls(all = TRUE))
```

Read the data from `TRABREV3.txt` into `vegdat`, set `header=T` to include the original column 
(header) names

```{r}
vegdat <- read.table("~/Downloads/TRABREV3.txt", header = TRUE)
```

We will check the dimension of the table. We should get _145_ rows and _27_ columns.

```{r}
dim(vegdat) # 145 plots (one per row) and 27 variables.
#yes, we get 145 rows and 27 variables. 
```

Here we are just exploring a bit more about this data object

```{r}
# class(vegdat) # what type of object is it?
# str(vegdat)  # what is in the object?
names(vegdat)  # print out the column (i.e., variable) names
#we have things like site, point, rank, vesp, star, etc. 

head(vegdat, 5)  # print out the first few lines of data
```
I'm not sure what hola, vesp, or star are in this context (bird abundance), but I get the idea. 

Remembering that: 

* `mayvgcov` and `mayvght` (% cover and height of vegetation in May) 
* `junvgcov and `junvght` (% cover and height of vegetation in June); 

* and horizontal cover variables, specifically: `baregrnd`, `bryopht`, `forbs`, `grass`, `rocks`, `shrubs`, and `trees` all as __percents__.


#### PART 2: Subset the variables for the FA into a different object
Variables that will make up `xdata` are: `mayvgcov`,`mayvght`,
`junvgcov`,`junvght`,`baregrnd`,`bryopht`,`forbs`, `grass`, 
`rocks`, `shrubs`, and `trees`.

```{r}
xdata <- subset(vegdat,select = c(mayvgcov, mayvght, junvgcov, junvght,
     baregrnd, bryopht, forbs, grass,
     rocks, shrubs, trees))
```

Some simple checking of  `xdata`. We should still get _145_ rows, but this time only _11_ columns.

```{r}
names(xdata)
class(xdata)
dim(xdata) # 145 rows and 11 columns
```
Yep, 145 rows and 11 columns. 

#### PART 3:  Basic stats and corelation matrix for the xdata

We will get basic stats using `stat.desc()` from the package `pastecs.`
Make sure that you have downloaded the package. You can install it 
with `install.packages("pastecs")`. Uncomment the chunk below and run to see
the summary of key statistics.

```{r}
#install.packages("pastecs")

#library(pastecs) # won't work if you have not installed the package first!!
#round(stat.desc(xdata), 2) # get some basic descriptive stats.
summary(xdata) 
```

Here is the correlation matrix rounded to the 2nd decimal place.

```{r}
round(cor(xdata), 2) # correlation matrix with elements rounded to 2 decimals
```
Initial thoughts on this: 

These correlation matrices reveal big patterns with vegetation data. The strongest relationships occur with the variables describing vegetation cover and height in May (0.92) and June (0.88). These variables are thus all highly positively correlated, indicating that they capture some common underlying gradient in vegetation structure. Sites that have greater vegetation cover early in the season also tend to have taller vegetation and maintain higher cover later in the growing season. This suggests that these measurements are all reflecting a similar ecological phenomenon related to site productivity and vegetation density.

May vegetation cover, May vegetation height, June vegetation cover, and June vegetation height are all strongly positively correlated with each other (May cover and May height = 0.92; June cover and June height = 0.88). These relationships indicate that sites with dense vegetation tend to maintain that structure across the growing season. Together, these variables likely represent a single dominant gradient of overall vegetation productivity or structural density.

Bare ground represents the opposite end of the vegetation gradient. Bare ground is moderately to strongly negatively correlated with vegetation cover and height (approximately −0.38 to −0.44) and strongly negatively correlated with trees (−0.62). This indicates that sites with greater vegetation structure tend to have less exposed soil, suggesting a tradeoff between vegetation density and open ground.

Grass shows one of the strongest relationships in the matrix with bare ground (−0.82). This suggests that where grass cover is high, exposed soil is minimal. Grass also shows moderate positive correlations with vegetation cover variables, indicating that grass contributes to overall vegetation cover at many sites.

Rock cover is positively correlated with bare ground (0.45) and negatively correlated with vegetation cover and height (roughly −0.29 to −0.36). This pattern suggests that rocky sites tend to support less vegetation and more exposed substrate.

Tree cover shows moderate positive correlations with vegetation cover and height (about 0.23–0.32) and a strong negative correlation with bare ground (−0.62). This suggests that sites with more trees tend to support greater vegetation structure and less exposed ground.

Bryophytes, forbs, and shrubs show weak relationships with other variables, typically falling between about −0.20 and 0.20. For example, bryophytes have correlations of −0.21 with May vegetation cover, −0.16 with May vegetation height, −0.17 with June vegetation cover, and −0.18 with June vegetation height. Bryophytes also show almost no relationship with forbs (0.00) and only a very weak relationship with rocks (−0.03). The largest correlation involving bryophytes is with trees (−0.28), which is still only modest compared with the stronger relationships observed among the main vegetation structure variables.

Overall, the correlation matrix suggests that the dominant pattern in the dataset reflects a gradient from densely vegetated sites characterized by greater plant cover, height, and tree presence to more open sites characterized by bare ground and rocky substrate. Other vegetation types appear to contribute smaller, more localized sources of variation within this broader structural gradient.


#### PART 4: Perform PCA using `princomp()` 

First, we do PCA to look at how many factors to keep to explain the majority
of the variances of X variables.  This is not strictly needed for Factor Analysis since
you may set the number of factors based on what might be possible to interpret 
but still useful to justify the number of factors to be used. 

**NOTE:** `cor` and `covar` in princomp( ) are not the same as `principal()` or `fa()` !!!

Here, `cor=TRUE` means using the correlation matrix (aka standardized X variables).

```{r}
veg.pca <- princomp(xdata, cor = TRUE)
```

Let's get a summary with `summary()`

```{r}
summary(veg.pca, loadings = TRUE, cutoff = 0.0)
```

Interpretation of the `summary()` outputs:

* The first part shows the standard deviation (eigenvalues), how much each individual
PC contribute to the variance, and the cumulative variance when you 
retain 1, 2, 3, ... principal components

* The second part shows the "loadings". This is in fact the __eigenvectors__. However, when we work with factor analysis later on with the function `principal()`, this will not be the case.

####

__General interpretation:__

The first section are the SDs and variance, while the second are the loadings (eigenvectors), or the direction of each principal component in relation to the original variables. 

The first principal component (Comp.1) has a standard deviation of 2.08 and explains approximately 39.3 percent of the total variance, making it the dominant axis of variation in the dataset. The second component (Comp.2) explains 16.1 percent, and the third component (Comp.3) explains 11.9 percent of the variance. Together, the first three components account for about 67.3 percent of the total variance, suggesting that most of the structure in the data can be summarized by these three axes. Adding the fourth component increases the cumulative variance explained to about 77.2 percent, and the first six components together account for roughly 91 percent of the total variance.

For Comp.1, the largest positive loadings occur for the vegetation structure variables: May vegetation cover (0.411), May vegetation height (0.409), June vegetation cover (0.417), and June vegetation height (0.367). Grass (0.291) and trees (0.234) also load positively, while bare ground (−0.354) and rocks (−0.251) load negatively. This indicates that the first component represents a gradient from densely vegetated sites with greater plant cover and height to more open sites characterized by exposed ground and rocky substrate. This interpretation aligns closely with the patterns observed in the correlation matrix.

The second component (Comp.2) reflects a somewhat different pattern. Bare ground (0.379) and shrubs (0.484) load positively, while grass loads strongly negatively (−0.531). Vegetation cover and height variables show smaller positive loadings. This component may represent variation in ground cover composition, distinguishing areas dominated by shrubs and exposed soil from those dominated by grasses.

The third component (Comp.3) appears to capture variation associated with certain vegetation types. Bryophytes load strongly positively (0.554), while trees load strongly negatively (−0.594) and forbs moderately negatively (−0.346). This suggests that this component may represent a composition gradient involving bryophytes versus woody vegetation or other plant groups, reflecting differences in site conditions or microhabitats.

Overall, the PCA results suggest that most of the variation in the dataset is structured along a dominant gradient of vegetation density and openness, with additional components capturing differences in ground cover composition and specific plant groups.

#####


Get the correlation matrix for X variables and principal components.
We do not need to scale the X variables since the correlation matrix already does that.
Notice that the `loadings` obtain here is very different from the so called "loadings"
above from `summary(veg.pca, loadings=TRUE,cutoff=0.0)`.

```{r}
loadings <- round(cor(xdata, veg.pca$scores), 7)
loadings
```
##------ 

## Quick interpretation: ##

This output shows the correlations between the original vegetation variables (X variables) and the principal components. Because PCA was based on the correlation matrix, the variables are already standardized, so no additional scaling is necessary. The values in the table represent how strongly each original variable is correlated with each principal component. These correlations are often used to interpret what ecological gradient each component represents.

It is important to note that these correlations differ from the “loadings” shown earlier by summary(veg.pca, loadings = TRUE). In the summary output, the values reported are the eigenvectors, which describe the mathematical direction of each principal component. In contrast, the table shown here contains the correlations between variables and components, which are often more intuitive for ecological interpretation.

Looking at the results, Comp.1 shows the strongest relationships with the main vegetation structure variables. May vegetation cover (0.85), May vegetation height (0.85), June vegetation cover (0.87), and June vegetation height (0.76) all correlate strongly and positively with this component. Grass (0.61) and trees (0.49) also show positive correlations, while bare ground (−0.74) and rocks (−0.52) show strong negative correlations. This indicates that the first principal component represents a gradient from densely vegetated sites with tall plant cover to open sites characterized by bare ground and rocky substrate.

The second component (Comp.2) reflects differences in ground cover composition. Bare ground (0.50) and shrubs (0.64) correlate positively with this component, while grass shows a strong negative correlation (−0.71). This suggests that Comp.2 distinguishes areas dominated by shrubs and exposed soil from areas dominated by grasses.

The third component (Comp.3) highlights variation among specific plant groups. Bryophytes correlate strongly positively (0.63), while trees correlate strongly negatively (−0.68) and forbs moderately negatively (−0.40). This suggests that Comp.3 may represent a vegetation composition gradient involving bryophytes versus woody vegetation or other plant types.

Overall, these correlations reinforce the interpretation that the first principal component captures the dominant vegetation density gradient in the dataset, while the second and third components describe additional variation related to vegetation composition and ground cover types.





##-----

Let's calculate the variances, percent variances, and cumulative percent variances and put these together using bind. round() these using d=2 (i.e., two decimal places)

```{r}
round(cbind("Variance" = veg.pca$sdev^2,
 "%" = 100*veg.pca$sdev^2 / sum(veg.pca$sdev^2),
  "Cumulative %" = 100 * cumsum(veg.pca$sdev^2) / sum(veg.pca$sdev^2)),
   d = 2)
```



Check the sum of the variances to make sure they sum up to __11__, which is the
number of X variables.

```{r}
sum(veg.pca$sdev^2) # check the sum of variances.
```

Get a scree plot with `sdev^2` (that is the variance or eigenvalue) of each component.

```{r}
plot(veg.pca$sdev^2, type = "b", pch = 16, xlab = "Component number",
ylab = "Eigenvalue", main = "Scree plot for x-variables of veg data")
```

#Interpretation so far -----

The results show that the first principal component explains the largest portion of the variation, with a variance of 4.33, accounting for 39.33% of the total variance. The second component explains 16.08%, and the third explains 11.91%. Together, the first three components account for 67.31% of the total variation, indicating that much of the structure in the vegetation data can be summarized with just a few axes. Including the fourth component increases the cumulative variance explained to 77.24%, and the first six components together explain about 91.18% of the total variance. After this point, each additional component contributes only a very small amount of additional variation.

The scree plot visualizes this pattern by plotting the eigenvalues of each component. The plot shows a steep drop between the first and second components, followed by a gradual flattening of the curve. This pattern indicates that the first component captures a major underlying gradient in the data, while later components represent progressively smaller sources of variation. The curve begins to level off around components three or four, suggesting that only the first few components capture meaningful structure in the dataset, while the remaining components mostly reflect minor variation or noise.

## ------


Graph PC 1 and 2 (i.e., dimensions 1 and 2) with a biplot.

```{r}
biplot(veg.pca)
```

**OPTIONAL:** These are alternative graphics

```{r, eval=FALSE}
plot.new() # start a new plot
plot(loadings[, 1], loadings[, 2], axes = FALSE,
xlim = c(-1, 1), ylim = c(-1, 1), xlab = "", ylab = "")
mtext(expression("Comp.2"), 1, las = 0) # put this on axis 1 (x-axis)
mtext(expression("Comp.1"), 2, las = 0) # put this on axis 2 (y-axis)
axis(1, at = c(-1, 1), pos = 0) # axes labels at 0, from -1 to 1
axis(2, at = c(-1, 1), pos = 0) # axes labels at 0, from -1 to 1
# labels for the x variables to each correlation value, pos=4 is to the right.
text(loadings[, 1], loadings[, 2], labels = colnames(xdata), pos = 4, cex = 0.7)
```
##----

##Interpretation so far: 

In the biplot, the direction and length of each arrow show how strongly each variable influences the position of sites in the ordination space. Sites located in the direction of a particular arrow tend to have higher values for that variable. For example, sites positioned toward the right side of the plot tend to have higher vegetation cover, height, and grass or tree presence, while sites toward the left side tend to have more bare ground and rock cover.

In the second plot, along Component 1, the vegetation structure variables (May vegetation cover, May vegetation height, June vegetation cover, and June vegetation height) cluster strongly on the right side. Grass and trees also fall on this side. In contrast, bare ground and rocks lie on the left side, indicating that these variables are negatively associated with the vegetation structure variables. This confirms that Component 1 represents a gradient from densely vegetated sites to open sites characterized by exposed soil and rocky substrate.

##----


#### PART 5: Factor analysis using two factors based on PCA analysis. 
##### General information:
The function `factanal()` can be used. 
Options for `rotation=` parameter  are `"varimax"`, `"promax"`, and `"none"`. 

The function principal() can be used. 

Another alternative is the `psych` package and the function `fa()`.
There are also options for getting the factor scores. 
`principal()` is used here from the `psych` package
factor scores will be calculated using the default "regression" method. 

To use the variety of rotations in `fa()` or in `principal()`, you also need the
package `GPArotation`
Rotations options: 

* `"none"`, `"varimax"`, `"quartimax"`, `"bentlerT"`, `"equamax"`, `"varimin"`, 
`"geominT"` and `"bifactor"` are orthogonal rotations. 

* `"promax"`, `"oblimin"`, `"simplimax"`,`"bentlerQ"`, 
`"geominQ"` and `"biquartimin"` and `"cluster"` are possible 
oblique transformations of the solution.

**NOTE:** `cor` and `covar` in `princomp()` are not the same as in
`principal()` or `fa()`!!!

In `principal()` and in `fa()`, `cor="cor"` means __Pearson's correlations__
but then `covar=FALSE` means using the correlation matrix!!

For `veg.fac1`, no rotation used. For the following codes to work, please make sure you have downloaded the `psych` and `GPArotation` packagesvia `install.packages("GPArotation")` and `install.packages("psych")`.

```{r}

#install.packages("GPArotation")
#install.packages("psych")

library(GPArotation)
library(psych)

veg.fac1 <- principal(xdata, nfactor = 2, rotate = "none",
      method = "regression", scores = TRUE, covar = FALSE, cor = "cor")
```
**NOTE:** `covar=FALSE` meaning using the correlation matrix. `cor="cor"` means
using __Pearson's correlations__ (there are other options!)

We can explore this object and its components via the functions `class()` and `str()`.
You can uncomment the codes to explore it by yourself by running this code chunk. We
will not run it here at the meantime because the outputs are quite lengthy.

```{r}
# uncomment and run to see ouputs

class(veg.fac1)   # what type of object is created?
str(veg.fac1)     # what do we get?
```

```{r}
veg.fac1
```

##### Let's go over some important components of this object:

```{r}
veg.fac1$loadings
```
`loadings` are simple __correlations between PC scores and x variables__, which is __NOT like using__ `princomp()`!
**Note:** very low correlations are not given and will appear missing.

```{r}
veg.fac1$weights
```

`weights` is the W (multipliers) to calculate factor scores

Next, we explore `communality` and `uniqueness`

```{r}
veg.fac1$communality # the h2 in the table
```

```{r}
veg.fac1$uniqueness  # the u2 in the table
```

We learned that the variance of each X variable is divided into __common variance (communality)__
and __unique variance (uniqueness)__

* `communality` is the __common variance__ of the X variables. It is the variance
_accounted for by the Factors_. In other words, `communality` tells you
what proportion of the variable's variance is a result of either:
  + The principal components
  + The correlations between each variable and individual factors

It is calculated by squaring each factor loading and 
then adding them up for each X variable (sum of squared factor loadings)

* `uniqueness` is the __unique variance__ of the X variables. It is, as its name implied, 
the variance _not accounted for by the Factors_, thus are _unique_ to each X variable.

It is *1 - communality* since the correlation matrix was used. In other words, it is 
a measure of how well the regression estimate the factor scores using the x variables 
as inputs based on root mean square residuals. A smaller number means a better fit.

**Note:** `fit` is a measure of how well the factors represent the full correlation (or
covariance matrix) of the X's. A smaller value means a better fit.

```{r}
veg.fac1$Structure
```

`Structure` is the Factor Structure matrix, which in this example represents the __simple correlations__
between each X variable and each Factor. If the factors are __uncorrelated (orthogonal)__
and __standardized__, or if is is an __orthogonal rotations of the axes__, 
the Factor Structure matrix and the Factor Pattern matrix is the same.

```{r}
veg.fac1$complexity
```

`complexity` is the Hoffman's index of complexity.

**Extra:** it represents the number of latent components needed to account for the observed variables. 
Whereas a perfect simple structure solution has a complexity of 1 in that each item would only load on one factor, a solution with evenly distributed items has a complexity greater than 1.
[Link to source](https://easystats.github.io/parameters/reference/model_parameters.principal.html#details)

Here, we calculate the __mean complexity__ of the variables.

```{r}
mean(veg.fac1$complexity)
```

#### Correlations between the factors and the X variables

```{r}
round(cor(xdata, veg.fac1$scores), 2)
```

This is same as `veg.fac1$loadings`. Smaller correlations are not shown.

##### Correlations between the two factors

```{r}
round(cor(veg.fac1$scores, veg.fac1$scores), 2)
```

##------

#Summary and interpretation so far
Here, you moved from PCA into a two factor solution using the psych package, motivated by the scree plot and the dominance of the first two components. Using principal(xdata, nfactors = 2, rotate = "none", covar = FALSE, cor = "cor", scores = TRUE), you fit two orthogonal factors based on the Pearson correlation matrix and computed regression factor scores. The factor loadings show Factor 1 capturing the vegetation structure gradient, with strong positive loadings for May and June cover and height and strong negative loadings for bare ground and rocks. Factor 2 captures the secondary composition contrast, with shrubs and bare ground loading positively and grass loading strongly negatively. You then examined communalities and uniquenesses to see which variables are well explained by the two factor model. Vegetation structure variables, grass, and bare ground have high communalities, meaning the two factors explain much of their variance, while bryophytes and forbs have very high uniqueness, meaning they are poorly captured by this two factor structure and likely reflect more localized or independent variation. This sets you up to decide whether rotation would make interpretation cleaner, and whether two factors are adequate for your goals versus adding a third factor to better represent those weaker, more independent plant groups.

##-----


##### "Hand calculate" the scores using the weights coefficients. 

Get the standardized xdata first

```{r}
Xscaled <- scale(xdata, center = TRUE, scale = TRUE)
head(Xscaled)
```

Check the dimension of the `weights` matrix

```{r}
dim(veg.fac1$weights)
```

```{r}
myscores <- Xscaled %*% veg.fac1$weights
head(myscores)
```

```{r}
head(veg.fac1$scores)
```

Calculating communality and uniqueness for the PC orthogonal rotation and for
other orthogonal rotations. Using the correlation matrix, the total variance
of the X's is 11 (=1 for each X since this is like you are using standardized x's)
can calculate the variances accounted for by squaring and adding the loadings. 

##### Using just PC1, you get the variance of all X's accounted for by PC1 only.

```{r}
PC1 <- veg.fac1$loadings[1:11, 1] # column 1
PC1

sum(PC1^2)
```

Here, PC1 accounts for 4.32614 of the 11 x-variables variances , which is its `communality`.

##### Using PC2, you get the variance of all X's accounted by PC2 only

```{r}
PC2 <- veg.fac1$loadings[1:11, 2] # column 2.
PC2

sum(PC2^2)
```

Again, PC2 accounts for 1.7683 of the 11 x-variables variances.

If you add PC1 and PC2 as the only two factors retained, you get the 
total communality, all x's and all factors. Total uniqueness not
accounted for by these two components is the number of x-variables minus
total communality. This is because the correlation matrix was used and, the
sum of correlation is 11 (diagonal entries of correlation matrix)

The total communality is:
```{r}
sum(PC1^2) + sum(PC2^2)
```

Thus, the uniqueness is:

```{r}
11 - (sum(PC1^2) + sum(PC2^2))
```

##### Then, how much communality of each X variable versus uniqueness?

```{r}
commonbyX <- PC1^2 + PC2^2
commonbyX
```

You may notice that `commonbyX` is same as `h2` column in the `veg.fac1` summary of outputs.
`h2` tells you how much each x-variable contribute to the common variance (`communality`).
It should sum up to 6.0944, which is the total communality calculated with the principal
components earlier. Let's check:

```{r}
sum(commonbyX)
```

It's the same number as total communality! 

##### How much is unique for each variable?

```{r}
uniquebyX <- 1 - (PC1^2 + PC2^2)
uniquebyX
sum(uniquebyX)
```

Similarly, you may notice that `uniquebyX` is similar to `u2` from the `veg.fac1`
summary of outputs. This corresponds to the uniqueness of the x-variables. Again,
it should sum up to 4.9055, which is the value of uniqueness calculated with 
the principal components earlier.

And finally, `commonbyX` and `uniquebyX` should sum up to 11, which is the number of
x variables, similar to the result we have above.

```{r}
sum(commonbyX) + sum(uniquebyX)
```

##Summary and interpretation so far ------

First, you standardized the X variables with scale(). That matters because your factor model is based on the correlation matrix, which treats each variable as having variance 1. You then confirmed the weights matrix is 11 by 2, meaning there is one regression weight per variable per factor. Multiplying the standardized data by these weights (Xscaled %*% veg.fac1$weights) reproduced the factor scores exactly, matching veg.fac1$scores. That is the key takeaway: the scores are not mysterious outputs, they are a weighted combination of standardized variables.

Next, you hand computed how much variance the retained factors explain. Squaring and summing the loadings for PC1 gave 4.326, and for PC2 gave 1.768. Those are the eigenvalue like sums of squared loadings for each factor, and they tell you how much of the total standardized variance each factor accounts for. Since total variance is 11, the two factor solution explains 6.094 out of 11, leaving 4.906 as residual or unique variance not explained by the two factors.

Finally, you broke that down by variable. For each X variable, the communality is PC1 squared plus PC2 squared, which matches the h2 column in the output. High communalities mean the two factor model captures most of that variable’s variance. In your results, the vegetation structure variables and bare ground and grass are well captured, with communalities around 0.74 to 0.87. In contrast, bryophytes and forbs have very low communalities, about 0.13 and 0.12, meaning most of their variance is uniqueness and not represented well by just two factors. That is the warning flag before rotations: the two factor solution is doing a good job summarizing the main vegetation structure gradient, but it is not doing much for bryophytes and forbs, which may require a different factor structure or an additional factor if you want those groups represented.

##------



#### `Varimax` rotation: A further orthogonal rotation

We will called PCs as RC1 and RC2 now. Remember to install the `psych` package first
if you have not done so already. In this example we will use the __varimax__ rotation
by setting the parameter `rotate = "varimax"`

Varimax __maximize variation in squared factor loadings__. Rotate so that the variation
of the squared factor loadings is maximized. Maximum variation occurs when factor
loadings are separated into being either close to 0 or close to 1 and -1.
.
```{r}
library(psych)
veg.fac2 <- principal(xdata, nfactor = 2, rotate = "varimax",
  method = "regression", scores = TRUE, covar = FALSE, cor = "cor")

veg.fac2
```

Let's check the components of `veg.fac2`

`communality` or `h2` in the table:

```{r}
veg.fac2$communality # the h2 in the table
```

`uniqueness` or `u2` in the table:

```{r}
veg.fac2$uniqueness  # the u2 in the table
```

`loadings` are the correlation between the original x variables and RC1 & RC2

```{r}
veg.fac2$loadings    # RC1 and RC2 values. NOTE: Very low correlations are not given.
```

`weights` are the multipliers

```{r}
veg.fac2$weights
```

`Structure` is the same as the `loadings` from the function `princomp()`,
which are the correlations of the x variables and the PCs, since is an orthogonal rotation.

```{r}
veg.fac2$Structure # same as loadings which ARE correlations for princomp()
```

`complexity` is Hoffman's index of complexity. The mean of complexity is shown in the
summary as well.

```{r}
veg.fac2$complexity  
mean(veg.fac2$complexity)
```

##### Correlations between the factors and the X variables

Let's find the correlations between the X variables and the factor scores
This will be the same as the `veg.fac2$loadings`. You may also notice that it is 
similar to `veg.fac2$Structure` as well, due to the fact that varimax is an 
__orthogonal rotation__.


```{r}
round(cor(xdata, veg.fac2$scores), 2)
```


##### Correlations between the two factors

The correlations between the two factors should be 0, since they are from an 
orthogonal rotation and hence are `uncorrelated`. Let's verify that fact:

```{r}
round(cor(veg.fac2$scores, veg.fac2$scores), 2)
```

##### "Hand calculate" the scores using the weights coefficients.

First, we get the standardized x variables

```{r}
Xscaled <- scale(xdata, center = TRUE, scale = TRUE)
```

Calculate with matrix multiplication. The results are the same

```{r}
myscores <- Xscaled %*% veg.fac2$weights
head(myscores)
head(veg.fac2$scores)
```

Using just RC1, you get the variance of all X's accounted for by factor1 only.

**NOTE:**  This works again since this is an orthogonal rotation.

##### RC1 and RC2 are independent (at right angles)

```{r}
RC1 <- veg.fac2$loadings[1:11, 1]
sum(RC1^2)
```

We can see that RC1 accounts for 3.275 of the variance.

##### Using RC2, you get the variance of all X's accounte by RC2 onlysum(RC2^2) 

```{r}
RC2 <- veg.fac2$loadings[1:11, 2]
sum(RC2^2) # How much does RC2 account for?
```

Similarly, we can see that RC2 account for 2.819 of the variance

##### Total communality and uniqueness. 

Still 2 factors so the same as the "none" rotation.

```{r}
sum(RC1^2) + sum(RC2^2) # total communality
11 - (sum(RC1^2) + sum(RC2^2)) # uniqueness
```

##### How much communality of each X variable versus uniqueness?

This is same as `h2` in the `veg.fac2` summary of outputs:
```{r}
commonbyX <- RC1^2 + RC2^2
commonbyX
sum(commonbyX)
```

##### How much is unique for each variable? What is left of the variance?

And this is the same as `u2`:

```{r}
uniquebyX <- 1 - (RC1^2 + RC2^2)
uniquebyX
sum(uniquebyX)
```

#Summary and intepretation so far: 

This section applies a varimax rotation, which is an orthogonal rotation used to make factor structures easier to interpret. Varimax works by maximizing the variation in squared factor loadings so that variables tend to load either strongly or weakly on a factor rather than moderately on several factors. In practice, this pushes loadings closer to 0 or ±1, producing a clearer separation of variables among factors. After rotation, the two factors are referred to as RC1 and RC2, but they still represent the same total variance as the unrotated solution because orthogonal rotations only redistribute variance between factors rather than changing the total explained variance.

Key numerical results

The rotated solution produces two factors with the following variance contributions:

RC1 variance: 3.275

RC2 variance: 2.819

Total variance explained: 6.094 of 11 variables

Proportion of total variance explained: 0.554 (55.4%)

Because the rotation is orthogonal, the two factors remain uncorrelated (correlation = 0), which was confirmed by the factor score correlation matrix. The communality and uniqueness values also remain identical to the unrotated solution since rotation redistributes loadings but does not change how much total variance each variable shares with the factor model.

Interpretation of the rotated factors

The first rotated factor (RC1) primarily represents overall vegetation structure and seasonal vegetation growth. Several vegetation cover and height variables load strongly on this factor:

junvgcov: 0.89

junvght: 0.84

mayvgcov: 0.81

mayvght: 0.80

These strong positive loadings suggest RC1 reflects a gradient of dense, tall vegetation during the growing season. Sites with high RC1 scores likely have greater vegetation coverage and height across both May and June.

The second rotated factor (RC2) captures a different ecological gradient related to ground surface composition. Strong loadings include:

grass: 0.93

bare ground: -0.86

shrubs: -0.50

trees: 0.49

This factor contrasts areas dominated by grass cover with areas characterized by bare ground or woody vegetation. High RC2 values therefore correspond to grass-dominated surfaces, while negative values indicate more exposed soil or shrub cover.

Variables poorly explained by the factors

Several variables have low communalities, meaning the two-factor model does not capture much of their variance:

bryophytes: h² = 0.13

forbs: h² = 0.12

rocks: h² = 0.30

trees: h² = 0.29

These variables remain largely unique or independent of the dominant vegetation gradients, suggesting they either vary independently of the main vegetation structure or would require additional factors to be well represented.

Overall interpretation

The varimax rotation clarifies the ecological meaning of the factors. The analysis suggests two main underlying gradients in the vegetation dataset:

Vegetation growth and canopy structure represented by RC1

Ground cover composition (grass versus bare or woody surfaces) represented by RC2

Together, these two factors explain just over half of the total variance (55%) in the vegetation variables. The rotation makes these patterns more interpretable by concentrating strong relationships within specific factors while leaving unrelated variables with low loadings.

##-----



#### veg.fac3: `Varimin` rotation: A further orthogonal rotation.

`varimin`  minimizes variance of factor loadings across factorized variables.

```{r}
veg.fac3 <- principal(xdata, nfactor = 2, rotate = "varimin",
        method = "regression", scores = TRUE, covar = FALSE, cor = "cor")
veg.fac3
```

As usual, let's explore the component of `veg.fac3`

`communality` or `h2`:

```{r}
veg.fac3$communality
```

`uniqueness` or `u2`:

```{r}
veg.fac3$uniqueness
```

`loadings` are the corelation between the RCs and the original x variables

```{r}
veg.fac3$loadings
```

`weights` are the multipliers to compute the factor scores:

```{r}
veg.fac3$weights
```

`Structure` is the same as `loadings`, which are the correlations: 

```{r}
veg.fac3$Structure
```

`complexity` is Hoffman's index of complexity

```{r}
veg.fac3$complexity
mean(veg.fac3$complexity)
```


##### Correlations between the factors and the X variables

Still the same as `veg.fac3$loadings ` and `veg.fac3$Structure` since this 
is still an orthogonal rotation.

```{r}
round(cor(xdata, veg.fac3$scores), 2)
```

##### Correlations between the two factors

The two factors should still be uncorrelated

```{r}
round(cor(veg.fac3$scores, veg.fac3$scores), 2)
```

##### "Hand calculate" the scores using the weights coefficients. 

Get the standardized xdata first

```{r}
Xscaled <- scale(xdata, center = TRUE, scale = TRUE)
```

The factor scores is the result of matrix multiplication between the
scaled x matrix and the weights matrix. The final matrix should be the
same as calling `veg.fac3$scores`.

```{r}
myscores <- Xscaled %*% veg.fac3$weights
head(myscores)
head(veg.fac3$scores)
```

##### Using just RC1, you get the variance of all X's accounted for by RC1 (factor 1) only.
**Note:**  This works again since this is an orthogonal rotation.
RC1 and RC2 are independent (at right angles)

```{r}
RC1 <- veg.fac3$loadings[1:11, 1]
sum(RC1^2)
```

##### Using RC2, you get the variance of all X's accounted by RC2 only

```{r}
RC2 <- veg.fac3$loadings[1:11, 2]
sum(RC2^2)
```

##### Total communality and uniqueness. 

Still 2 factors so it is the same as the "none" rotation.

Total communality:
```{r}
sum(RC1^2) + sum(RC2^2)
```

Uniqueness:

```{r}
11 - (sum(RC1^2) + sum(RC2^2))
```

##### Then, how much communality of each X variable versus uniqueness?

```{r}
commonbyX <- RC1^2 + RC2^2
commonbyX
sum(commonbyX)
```

##### How much is unique for each variable? What is left of the variance?

```{r}
uniquebyX <- 1 - (RC1^2 + RC2^2)
uniquebyX
sum(commonbyX)
```

## Summary and interpretation so far -----

This section applies a varimin rotation, which is another orthogonal rotation of the same two factor solution. Unlike varimax, which tries to push loadings toward extreme values near 0 or ±1, varimin minimizes the variance of the factor loadings across variables. In effect, it spreads loadings more evenly across factors rather than concentrating them. Importantly, because this is still an orthogonal rotation, the two factors remain uncorrelated, and the total amount of variance explained by the model does not change. The model still explains 6.094 of the 11 total units of standardized variance (55.4%), leaving 4.906 as uniqueness or unexplained variance.

Numerically, the rotation redistributes variance between the two factors. Under the varimin solution, RC1 explains 4.161 units of variance (37.8%), while RC2 explains 1.933 units (17.6%). This contrasts with the previous varimax solution, where variance was more evenly split between the factors. Here, the rotation concentrates more explanatory power into RC1, making it a stronger general factor while RC2 becomes more secondary.

Looking at the loadings helps clarify what these factors represent. RC1 now captures a broad vegetation and ground structure gradient. Several vegetation structure variables load strongly on it, including May and June vegetation cover and height (0.64 to 0.77). Grass also loads strongly and positively (0.77), while bare ground (−0.84) and rocks (−0.55) load negatively. This suggests RC1 reflects a general gradient from vegetated surfaces to exposed substrate. Sites with high RC1 scores tend to have dense vegetation and grass cover, while low RC1 scores correspond to bare ground or rocky surfaces.

The second factor (RC2) now represents a weaker secondary gradient. It captures variation related to vertical vegetation structure, with shrubs loading strongly (0.62) and June vegetation height and cover also showing moderate loadings (0.56 and 0.58). Grass loads negatively on this factor (−0.53), suggesting RC2 contrasts shrub or taller vegetation structure with grass dominated ground cover.

As in earlier analyses, some variables remain poorly represented by the two factor model. Bryophytes (h² ≈ 0.13), forbs (h² ≈ 0.12), and rocks (h² ≈ 0.30) have low communalities, meaning most of their variance is unique and not captured by the dominant vegetation gradients. This indicates that these vegetation types vary somewhat independently of the main structural patterns in the dataset.

Overall, the varimin rotation shows the same underlying dimensional structure as the earlier solutions but distributes the loadings differently. Instead of separating variables cleanly into distinct factors, it produces a solution where one dominant factor summarizes the general vegetation versus exposed ground gradient, while the second factor captures a weaker structural contrast among vegetation types.





#####  TODO: veg.fac4: Add in the code for `equamax` orthogonal rotation here (not graded)

Follow the structure of each examples above, add in the codes for `equamax` orthogonal rotation here. Make
sure you do not miss any calculations. Create additional markdown commentaries and code chunks if you wish!
This part is not graded, however the outputs will be asked in the upcoming TODO questions.

```{r}
#### veg.fac4: `Equamax` rotation: A further orthogonal rotation.

# Equamax is an orthogonal rotation that blends the goals of varimax and quartimax.
# It aims for a "middle ground" solution: simpler structure within factors while
# also avoiding one overly general factor dominating all variables.

library(psych)

veg.fac4 <- principal(
  xdata, nfactor = 2, rotate = "equamax",
  method = "regression", scores = TRUE, covar = FALSE, cor = "cor"
)

veg.fac4
```

```{r}
##### Explore key components of `veg.fac4`

# Communality (h2)
veg.fac4$communality

# Uniqueness (u2)
veg.fac4$uniqueness

# Loadings: correlations between the original x variables and RC1 & RC2
veg.fac4$loadings

# Weights: multipliers used to compute factor scores
veg.fac4$weights

# Structure: same as loadings here because equamax is an orthogonal rotation
veg.fac4$Structure

# Hoffman's index of complexity (and its mean)
veg.fac4$complexity
mean(veg.fac4$complexity)
```

```{r}
##### Correlations between the factors and the X variables

# Should match veg.fac4$loadings (and veg.fac4$Structure) because orthogonal rotation
round(cor(xdata, veg.fac4$scores), 2)
```
```{r}
##### Correlations between the two rotated factors

# Should be 0 (uncorrelated) for an orthogonal rotation
round(cor(veg.fac4$scores, veg.fac4$scores), 2)
```

```{r}
##### "Hand calculate" the factor scores using the weights coefficients

# Standardize X (consistent with correlation-matrix based solution)
Xscaled <- scale(xdata, center = TRUE, scale = TRUE)

# Matrix multiplication gives factor scores
myscores <- Xscaled %*% veg.fac4$weights

# Compare the first few rows to confirm they match
head(myscores)
head(veg.fac4$scores)
```
```{r}
##### Variance accounted for by each rotated component (RC1, RC2)

# Because equamax is orthogonal, we can still compute variance explained
# by squaring and summing loadings across variables.

RC1 <- veg.fac4$loadings[1:11, 1]
RC2 <- veg.fac4$loadings[1:11, 2]

sum(RC1^2)  # variance explained by RC1
sum(RC2^2)  # variance explained by RC2
```

```{r}
##### Total communality and uniqueness (two-factor solution)

total_communality <- sum(RC1^2) + sum(RC2^2)
total_uniqueness  <- 11 - total_communality  # total variance = 11 (cor matrix)

total_communality
total_uniqueness
```

```{r}
##### Communality and uniqueness by X variable

commonbyX <- RC1^2 + RC2^2
commonbyX
sum(commonbyX)   # should equal total_communality

uniquebyX <- 1 - commonbyX
uniquebyX
sum(uniquebyX)   # should equal total_uniqueness

# Sanity check: should sum to 11 variables worth of variance
sum(commonbyX) + sum(uniquebyX)
```

###Summary and Interpretation: 

This section applies an equamax rotation, which is another orthogonal rotation of the same two factor model. Equamax is designed as a compromise between varimax and quartimax style rotations. Rather than strongly concentrating loadings into a single factor or evenly spreading them across factors, it attempts to simplify the structure both across variables and across factors. Because this is still an orthogonal rotation, the two factors remain uncorrelated, and the total variance explained by the two factor model does not change. As in the previous solutions, the model explains 6.094 of the 11 total standardized variance units (55.4%), leaving 4.906 as uniqueness or unexplained variance.

Looking at the loadings, the first rotated component (RC1) clearly represents a vegetation structure gradient. Several vegetation variables load strongly and positively on RC1, including May vegetation cover (0.86), May vegetation height (0.85), June vegetation cover (0.92), and June vegetation height (0.86). These strong positive loadings indicate that RC1 reflects sites with dense and tall vegetation across the growing season. Negative loadings for bare ground (−0.38), rocks (−0.37), and bryophytes (−0.34) suggest that lower RC1 values correspond to sites dominated by exposed substrate or sparse vegetation.

The second rotated component (RC2) captures a different gradient related primarily to ground cover composition. Grass loads very strongly and positively on RC2 (0.92), while bare ground loads strongly in the opposite direction (−0.81). Shrubs (−0.56) and rocks (−0.40) also contribute to the negative side of this axis. This indicates that RC2 distinguishes grass dominated surfaces from areas characterized by bare ground, shrubs, or rocky substrate.

The communality values show how well the two factor solution explains each variable. Vegetation structure variables are well represented, with communalities around 0.74 to 0.87, meaning most of their variance is captured by the factors. Grass and bare ground are also strongly represented. In contrast, bryophytes (h² ≈ 0.13), forbs (h² ≈ 0.12), and rocks (h² ≈ 0.30) have low communalities, indicating that much of their variance remains unique and is not well explained by the two dominant gradients.

Overall, the equamax rotation reveals essentially the same ecological structure observed in the previous rotations but with a slightly clearer separation of the major gradients. The first factor summarizes overall vegetation density and seasonal growth, while the second factor captures ground cover composition, particularly the contrast between grass and bare or shrub dominated surfaces. Together these two factors describe the primary structural patterns in the vegetation dataset while leaving several minor vegetation types only weakly represented.

####-------









##### End of TODO

**TODO Question 1:** In all analyses, **two factors were selected**. Using a number of ways to determine how many factors to retain (as you did using PCA in Exercise 2), is this number
of factors justified for these data? 

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 1**

**TODO Question 2:** For the unrotated factor pattern `veg.fac1`, what is the loading value for Factor
1 with `mayvgcov`? What is the structure value for this also? Explain what these
values mean.

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 2**

**TODO Question 3:** Using orthogonal rotations only (`veg.fac1`, `veg.fac2`, `veg.fac3`, `veg.fac4`), what
orthogonal rotation (including no rotation) gave the best results?

a. Justify your choice as to why you believe these results are best for
achieving a simple and interpretable structure. Also indicate why others
were not chosen.

b. Use the factor structure (correlations between original variables and
Factors) to decide which variables relate to each factor. Remember to
state __what correlation cutoff__ you are using (e.g., |r|>= 0.5 or |r|>= 0.8);
and  __why__ you made this choice.

c. Add a possible label for each factor (e.g., vegetation structure might be
one factor). Use your answers from part b, but also consider whether
the correlations are positive or negative between a factor and the
original variables correlated with it.

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 3**

**TODO Question 4:** Based on the two factors retained and using your selected orthogonal rotation:

a. How much of the variability of each of the original X variables is
accounted for (communality):
* by each factor
* for both factors combined?

b. How much of the variability of each of the original X variables is NOT
accounted for (unique variance): 
* by each factor
* for both factors combined?

c. For all X variables combined, what are the communality and unique
variances:
* by each factor
* for both factors combined?

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 4**

**TODO Question 5:** Calculate the communality for `mayvgcov` for **Factor 1** using your selected
orthogonal rotation “by hand”. Confirm that you obtained the same
communality value as you reported in 4a for this variable and factor.

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 5**

**TODO Question 6:** Calculate the factor score for **Factor 1** for the first observation in the dataset,
using your selected orthogonal rotation. Confirm that this is the same as the
score calculated using `princomp()` in R.

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 6**

#### Oblimin rotation: An oblique rotation.
**Note:** we will call the factors as TC1 and TC2 now

Run the same analysis, but this time with `rotate="oblimin"`.
`oblimin` allows for the factors to be __correlated (not orthogonal)__.

```{r}
veg.fac5 <- principal(xdata, nfactor = 2, rotate = "oblimin",
  method = "regression", scores = TRUE, covar = FALSE, cor = "cor")

veg.fac5

# str(veg.fac5) uncomment this if you want to fully explore the object
```

`communality` and `uniqueness`:
```{r}
veg.fac5$communality
```

```{r}
veg.fac5$uniqueness
```

`weights` still contains the multipliers

```{r}
veg.fac5$weights # multipliers
```

Hoffman's index of complexity and mean complexity:

```{r}
veg.fac5$complexity  # the com in the table, Hoffman's index of complexity
mean(veg.fac5$complexity) # Hoffman's index of complexity also shown in the veg.fac5 summary
```

However, `loadings` is no longer the correlations between the TCs (factor scores) and the X variables.
Since _oblimin_ is an __oblique rotation__, this is now showing the __Factor Pattern__.

```{r}
veg.fac5$loadings # factor pattern
```

`Structure` is the correlations between the X variables and the factor scores.
Since this is an oblique rotation, you can see that Factor Pattern is different from
Factor Structure.

```{r}
veg.fac5$Structure
```

Let's verify that Structure is the simple correlations between the X variables and factor scores
by manually calculating with `cor()`.
```{r}
round(cor(xdata, veg.fac5$scores), 2)
```

We can can see that they are pretty much identical. And they are completely different from
`veg,fac5$loadings`, which is now the factor pattern.


##### Correlations between the two factors

Let's explore the correlation between the factors

```{r}
round(cor(veg.fac5$scores, veg.fac5$scores), 2)
```

We can see that with an oblique rotation like `oblimin`, the factors are
__correlated__, which means the axes are __not 90 degrees (orthogonal)__ from each other. Thus, 
we cannot interpret each factor separately.

Since the factors are correlated, the variance accounted for by Factor 1
overlaps with the variance accounted for by Factor 2.
We can calculate the two variances, but this will be too big.
Alternatively, we can calculate just the independent bits, but that will
be too small. This will be demonstrate later on in this code.

**TODO Question 7:** Using the oblique rotation (`veg.fac5`):

a. What is the correlation between the two factors? Is this “low enough”
for factors to be considered “nearly independent”?
b. Repeat what you did in question 3b and 3c.

_Type your answer here:_

```{r}
# your codes here (if applicable)
```

**End of Question 7**

##### "Hand calculate" the scores using the weights coefficients. 

Standardize the original X variables

```{r}
Xscaled <- scale(xdata, center = TRUE, scale = TRUE)
```

Matrix multiplication

```{r}
myscores <- Xscaled %*% veg.fac5$weights
```

Verify that they are indeed similar

```{r}
head(myscores)
head(veg.fac5$scores)
```

We can try to get the variance accounted for by each factor, but notice here that
they will overlap overlap since the factors are __correlated__ now.

Variance accounted for by TC1. Notice here that we are using `Structure` and not `loadings`
since `Structure` is the correlations matrix, and it is no longer the case for `loadings`.

```{r}
TC1 <- veg.fac5$Structure[1:11, 1]
sum(TC1^2)
```
what about TC2? 

```{r}
TC2 <- veg.fac5$Structure[1:11, 2]
sum(TC2^2)
```

```{r}
sum(TC1^2) + sum(TC2^2) # now greater than the sum of two factors for none or orthogonal
```

the factors overlap!!! So, there is "double-counting" of some of the variance explained.

##### What about the SS loadings?

```{r}
TC1 <- veg.fac5$loadings[1:11, 1]
sum(TC1^2) # How much does TC1 account for?
```

##### What about TC2? 

```{r}
TC2 <- veg.fac5$loadings[1:11, 2]
sum(TC2^2) # How much does TC2 account for?
```

```{r}
sum(TC1^2) + sum(TC2^2) # now less than the sum of two factors for none or orthogonal
```

the factors overlap!!! Using the SS "loadings", the overlapping part is removed.

##### What does `princomp()` report here?  

```{r}
veg.fac5
```

So, using `Structure`, we get the double-counting of variances:
TC1 (3.959523) + TC2 (3.835755) = 6.7953

Using `loadings`, we get the non-overlapping parts of variances:
TC1 (3.60252) + TC2 (2.36017) = 5.9269

Reported in the `veg.fac5` summary?
TC1 (3.67) + TC2 (2.43) = 6.1, which is how much is accounted for by 2 factors!!

Still 2 factors so the same as the "none" rotation.
As noted that is harder to see here... since there is overlap
of variance accounted for given these non-independent factors.

```{r}
TC1 <- veg.fac5$Structure[1:11, 1]
```

##### What about TC2? 

```{r}
TC2 <- veg.fac5$Structure[1:11, 2]
```

```{r}
commonbyX <- TC1^2 + TC2^2 # same as h2 in the veg.fac5 summary of outputs.
commonbyX # not really correct since factors are not independent!
sum(commonbyX)
```

##### How much is unique for each variable? What is left of the variance?

```{r}
uniquebyX <- 1 - (RC1^2 + RC2^2) #same as u2 in the veg.fac5 summary of outputs. 
uniquebyX # not really correct since factors are not independent!
sum(uniquebyX)
```

```{r}
sum(commonbyX) + sum(uniquebyX)
```

#### Which one to choose? 
We want a simple structure that is easy to interpret and hopefully label. 
We would like this to be from a orthogonal rotation if possible so factors are indepedent!

**TODO Question 8:** Overall, based on your analysis and interpretation, which rotation would you
recommend? State your choice and then provide a short paragraph to justify
your choice and why others were not recommended by you.

_Type your answer here:_

**End of Question 8**

### Congratulation!










