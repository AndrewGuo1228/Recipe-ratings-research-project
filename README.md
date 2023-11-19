# Exploring the Impact of Nutritional Values on Recipe Ratings
### Yanhao Guo

- Introduction
- Data cleaning
- Exploratory Data Analysis
- Assessment of Missingness
- Hypothesis Testing

---
## Introduction

The culinary world has witnessed a remarkable expansion, offering an extensive array of recipes and diverse cuisines. In this project, I embark on a detailed exploration of a rich dataset from food.com, comprising an assortment of recipes and their respective user ratings. Our primary goal is to unearth trends and patterns that shed light on the factors influencing a recipe's popularity, with a special emphasis on the impact of nutritional values on user ratings.

Two datasets, comprehensive collections from food.com, are used for this project: recipes and ratings. The recipes dataset is a vast compilation, containing 83,782 unique recipes. Each entry provides vital information, including the recipe's name, ID, preparation time, contributor ID, submission date, tags, detailed nutrition information, number of preparation steps, textual description of the steps, and a brief overview of the recipe. The ratings dataset is equally extensive, encompassing 731,927 individual reviews. Each review captures specific interactions with a recipe, featuring user ID, recipe ID, interaction date, the given rating, and the text of the review.

I have merged two datasets, then performed some data cleaning to prepare them ready for exploratory analysis. This initial phase aims to unravel the basic interrelationships within the dataset and pave the way for more in-depth inquiries. An other key focus of the study is the assessment of data missingness, the NMAR analysis concentrates on the 'review' column, endeavoring to understand the patterns behind the missing reviews. Apart from NMAR analysis, I also carried MAR analysis that focused on why rating data (which have 6.4% missing rate), to decipher the patterns and dependencies in the missing data. As I delve deeper into the project, my attention shifts to the nutritional aspect of the recipes, especially the caloric content, and its influence on recipe ratings. I examine whether the calorie count of a recipe has a significant impact on its popularity, as reflected in the user ratings. This aspect of the research is particularly relevant for understanding user preferences and guiding recipe creators in designing dishes that resonate with the audience's nutritional considerations.

---
## Data cleaning

> Merge Datasets: 

To start off, I have merged two datasets, recipes and ratings, based on a common identifier (recipe ID), and handled missing ratings by replacing zeros with NaN (Not a Number) values. I then calculated the average rating for each recipe and added this information to the recipes dataset. Finally, I addressed an issue with the user_id column being in float format by filling missing values with -1 and converting the column to integer type.

I received following information as a result: 

```
  name           object 
  id             int64  
  minutes        int64  
  contributor_id int64  
  submitted      object 
  tags           object 
  nutrition      object 
  n_steps        int64  
  steps          object 
  description    object 
  ingredients    object 
  n_ingredients  int64  
  average_rating float64
  user_id        int64  
  recipe_id      float64
  date           object 
  rating         float64
  review         object 
```
Before conducting analysis related to the datasets, we would first conduct data cleaning to make the data more convenient for analysis.
By looking at the above information, I noticed some data type that required changes

> Converge variable name from object into string type:

First, to ensuring that the 'name' column, which contains the names of the recipes, is consistently in string format. This conversion is essential for text processing and ensures that all recipe names are treated uniformly in subsequent analyses.

> Converge Object into list (tags, nutrition, steps, ingredients):

The second step I am going to do to the dataframe is the tags, nutrition, steps and ingredients columns. The four column all look like lists of string, but by checking the specific entry in the dataframe, we find that they are actually not lists. This could due to when web scraping, data collecter does not convert the text into list. As a result, we take action to convert the these three columns into list of string.

> Separating Elements of 'nutrition' into Individual Columns:

After converging the type of nutrition, now I am able to split the nutrition columns into different nutrition component (i.e. calories) in separate columns. By breaking down the 'nutrition' list into separate columns for each nutritional component, the dataset will be more accessible for analysis. This step is particularly relevant for investigating how specific nutritional factors, such as calorie content, influence recipe ratings.

> Converge submitted and date into datetime type:

The 'submitted' column contains dates when recipes were submitted. Converting this column to datetime format allows for easier manipulation and analysis of date-based data, such as filtering by year, month, or day, and time-series analysis. Similar to 'submitted', the 'date' column represents the dates of user interactions (ratings/reviews). Converting to datetime format standardizes these timestamps for any time-based analysis or filtering.

> Calculating and Adding Average Rating Per Recipe:

This step adds a crucial piece of data for the analysis – the average rating for each recipe. By calculating and appending this information, I enable an investigation into the relationship between various recipe features (like nutritional content, preparation complexity) and their overall user ratings. **Also, I believe that the reason why there are empty rating is that people do not fill in. As a result, I replace 0 with nan value**

> Recipe_complexity	columns:

**Purpose:** This column categorizes recipes into 'Simple' or 'Complex' based on the number of steps required to prepare them. Recipes with 10 or fewer steps are considered 'Simple', while those with more than 10 steps are deemed 'Complex'.
**Reason for Creation:** This categorization allows for an analysis of how recipe complexity influences user ratings. It's particularly useful for testing whether simpler recipes are more popular or rated higher than more complex ones.

> Calorie_range column:

**Purpose:** To group recipes into 'Low', 'Medium', and 'High' calorie ranges.
**Reason for Creation:** This column is created to facilitate the investigation of whether the calorie content of a recipe affects its popularity or average rating. I chose 600 and 800 calorie as the threshold because 1800/2400 are recommended daily calories for woman and man respectively, 600/800 would be recommended per meal. By categorizing the recipes into these calorie ranges, I can analyze trends and preferences related to caloric content.

> Prep_time_category:	

**Purpose:** To categorize recipes based on their preparation time into 'Quick', 'normal', and 'Time-Consuming'.
**Reason for Creation:** This categorization aims to explore how the time required to prepare a recipe influences its ratings. I chose 30 and 60 as the threshold to separate three categories as in tags, there're below 30 minutes tag and below 60 minutes tags. It allows for a nuanced analysis of user preferences towards quick-to-make versus more time-consuming recipes.

> Rating_missing:

**Purpose:** To create a binary indicator that flags whether a rating for a recipe is missing.
**Reason for Creation:** This column is critical for the analysis of missing data. It helps identify patterns in missing ratings, potentially revealing insights about why certain recipes may not have been rated and whether the missingness is random or systematically related to other recipe features.

After all these data cleaning and operations based on my observation, I received a cleaned and workable datasets with 29 columns:

```
 0   name                234429 non-null  object        
 1   id                  234429 non-null  int64         
 2   minutes             234429 non-null  int64         
 3   contributor_id      234429 non-null  int64         
 4   submitted           234429 non-null  datetime64[ns]
 5   tags                234429 non-null  object        
 6   nutrition           234429 non-null  object        
 7   n_steps             234429 non-null  int64         
 8   steps               234429 non-null  object        
 9   description         234315 non-null  object        
 10  ingredients         234429 non-null  object        
 11  n_ingredients       234429 non-null  int64         
 12  user_id             234429 non-null  int64         
 13  recipe_id           234428 non-null  float64       
 14  date                234428 non-null  datetime64[ns]
 15  rating              219393 non-null  float64       
 16  review              234371 non-null  object        
 17  calories            234429 non-null  float64       
 18  total_fat           234429 non-null  float64       
 19  sugar               234429 non-null  float64       
 20  sodium              234429 non-null  float64       
 21  protein             234429 non-null  float64       
 22  saturated_fat       234429 non-null  float64       
 23  carbohydrates       234429 non-null  float64       
 24  average_rating      231652 non-null  float64       
 25  recipe_complexity   234429 non-null  object        
 26  calorie_range       234327 non-null  category      
 27  prep_time_category  234426 non-null  category      
 28  rating_missing      234429 non-null  bool 
 ```
We will be using columns: **total_fat, minutes, n_steps, average_rating, calories, recipe_complexity, calorie_range, prep_time_category, rating_missing in particular,** so I will only present them selectively in dataframe below to keep things neat.

|   total_fat |   minutes |   n_steps |   average_rating |   calories |
|------------:|----------:|----------:|-----------------:|-----------:|
|          10 |        40 |        10 |                4 |      138.4 |
|          46 |        45 |        12 |                5 |      595.1 |
|          20 |        40 |         6 |                5 |      194.8 |
|          20 |        40 |         6 |                5 |      194.8 |
|          20 |        40 |         6 |                5 |      194.8 |

| recipe_complexity   | calorie_range   | prep_time_category   | rating_missing   |
|:--------------------|:----------------|:---------------------|:-----------------|
| Simple              | Low             | normal               | False            |
| Complex             | Low             | normal               | False            |
| Simple              | Low             | normal               | False            |
| Simple              | Low             | normal               | False            |
| Simple              | Low             | normal               | False            |

---
## Exploratory Data Analysis

### Univariate Analysis:

> Plot 1: Distribution of 'average_rating'

This histogram visualizes the distribution of average ratings for recipes. It helps to understand the general trend in how users rate recipes on the platform.

<iframe src="assets/average_rating_distribution.html" width=800 height=600 frameBorder=0></iframe>

The distribution of the ratings is clearly skewed to the left, indicating that ratings of recipe are most likley to be either 4 or 5

> Plot 2: Distribution of 'total_fat'

This histogram visualizes the distribution of the total fat content in recipes. It provides an understanding of how fat content varies across different recipes in the dataset.

<iframe src="assets/total_fat_distribution.html" width=800 height=600 frameBorder=0></iframe>

The distribution of the ratings is clearly skewed to the right, with the median value (red bar) significantly less than the maximum value (there are some distinctive outliers)

> Plot 3: Distribution of 'n_steps'

This histogram visualizes the distribution of the number of steps ('n_steps') involved in recipes. It offers an insight into the complexity of recipes in terms of their preparation steps.

<iframe src="assets/n_steps_distribution.html" width=800 height=600 frameBorder=0></iframe>

The distrubution is also right skewed distribution. The center for the graph is around 7, meaning most recipes have 7 steps. Also, we could observe the graph have a lot outliers that have very big step numbers. after observing this histogram, I decided to use 20 steps as the threshold to distinguish the simple recipe and complex recipe as most of recipe having steps less than 20 steps.

### Bivariate Analysis:

> Recipe Complexity vs Average Rating with a threshold of 20 steps:

The following box plot compares the average ratings of recipes categorized as 'Simple' and 'Complex' using a threshold of 10 steps:

- Recipe Complexity: Recipes with 20 or fewer steps are labeled 'Simple', while those with more than 20 steps are labeled 'Complex'.
- Average Rating: The distribution of average ratings for each complexity category is depicted in the plot.

  This analysis with the specific threshold provides insights into how the number of steps in a recipe, as a measure of complexity, might influence its average ratings from users. 

<iframe src="assets/Recipe_ComplexityvsAverage_Rating.html" width=800 height=600 frameBorder=0></iframe>

  We noticed that the rating distribution of recipe with steps more than 20 (complex recipe) is larger than that of simple recipe, bring me to the light that the complexity might be positively related with rating received by the recipes.


> Relationship between the calorie content of a recipe and its average user rating:

  Plotting calories on the x-axis and average ratings on the y-axis. This would visually reveal any patterns or trends between these two variables.

<iframe src="assets/caloriesVSaverage_ratings.html" width=800 height=600 frameBorder=0></iframe>

  By observing the scatter plot, we noticed that there're much fewer data we have for the high calories food, this might introduce bias to our prediction results, as clearly the high calories food are rated as 5. This might be a reasonable fact in reality: fewer people wanna try high-calories food, but whoever wanna try is very likely to like this kind of food originally so give a high rating afterwards. 

### Aggregates analysis

> Pivot Table by Preparation Time and Average Ratings:

I built a pivot table by grouping recipes by preparation time (such as quick or time-consuming, which we have done in the data cleaning), with calorie amount as value, and four statistics as columns: mean, median, minimum, maximum; 

|   mean_calories |   median_calories |   min_calories |   max_calories |
|----------------:|------------------:|---------------:|---------------:|
|         338.799 |             244.1 |            0   |        45609   |
|         432.154 |             331   |            0.2 |        18268.7 |
|         557.125 |             389.1 |            0   |        36188.8 |


I had a very interesting observation here. As the preparation time category goes from quick to time-consuming, both mean and median calories amount are increasing, there might exists a positive relationship between time taken for preparation and calories of the recipe. In order to see this clearer, I have visualized these aggregates analysis in a boxplot below:

<iframe src="assets/prep_time_categoryVScalories.html" width=800 height=600 frameBorder=0></iframe>

We can observe that is a difference between the distribution for three prepration time categories with more time-consuming categories having a larger calories distribution (higher median value), which might be an interesting research topic to carry hypothesis test forward.(Although I have chosen something else hhha, as I think this pattern is easy to observe, testing on something else will add more to this project)

---
## Assessment of Missingness

### NMAR Analysis

In the NMAR (Not Missing At Random) analysis, my attention centers on why reviews might be missing in the merged dataset, particularly focusing on the potential reasons behind users not leaving reviews. One plausible explanation for this absence of reviews is that some users may find the recipe straightforward and feel that there isn't much to comment on. Consequently, they might opt not to leave a review. This assumption suggests that the decision to skip the review process could be influenced by the perceived simplicity of the recipe.

To transition this scenario from an NMAR to a Missing At Random (MAR) situation, where the missingness depends on observed data rather than unobserved, we could consider incorporating an additional metric – a personal difficulty rating assigned by each user who tries the recipe. This rating would serve as observable data, potentially explaining the missingness in reviews. For instance, if users consistently do not leave reviews for recipes they rate as 'easy,' this pattern would support the hypothesis that missing reviews are linked to the perceived simplicity of the recipes, making the missingness MAR instead of NMAR.

### Missingness Dependency:

for this part, I have coded a permutation test function first, which takes in four arguments: (dataframe, column_to_test, missingness_indicator, n_permutations=1000); This function will return the p value and plot the histogram of permutation distribution. 
I have noticed that there're a significant missing percentage (6.41%) of rating, so I will focus on finding possible dependent factors for missing rating.
**1% Significance Level will be applied**

> Test the missing of ratings on minutes:

Now, we focus on the missingness of rating in the merged dataframe and test the dependency of this missingness to minutes:

- Null hypothesis: the distribution of the minutes when rating is missing is the same as the distribution of the minutes when rating is not missing 
- Alternative hypothesis: the distribution of the minutes when rating is missing is different from the distribution of the minutes when rating is not missing observed
- Statistics: the absolute difference between minutes' mean of these two distributions. 
I also draw distribution plots about these two distributions:

<iframe src="assets/minutes_distriburion_on_rate.html" width=800 height=600 frameBorder=0></iframe>

  I have performed the permutation for 1000 times and then plot the distribution of simulated results about the absolute difference. (red vertical line is the observed statistics) 

<iframe src="minutes_permutation_differences.html" width=800 height=600 frameBorder=0></iframe>

The p-value I received is 0.107. As a 0.05 as a significance threshold is used and 0.107 > 0.05, we fail to reject the null hypothesis that the rating is not dependent on the number of minutes. We can directly observe from the plot as well. Based on our test result, we can see that the missingness of the rating is MCAR because the missingness of rating is not correlated with the minutes (Strictly speaking, I will need to perform dependency test for all columns to reach this conclusion, but here's for simplicity, assuming this is the only column we need to take test on).

> Test the missing of ratings on n_steps:

Now, we focus on the missingness of rating in the merged dataframe and test the dependency of this missingness to number of steps:

- Null hypothesis: the distribution of the n_steps when rating is missing is the same as the distribution of the n_steps when rating is not missing 
- Alternative hypothesis: the distribution of the n_steps when rating is missing is different from the distribution of the n_steps when rating is not missing observed
- Statistics: the absolute difference between number of steps' mean of these two distributions. 
I also draw distribution plots about these two distributions:

<iframe src="assets/numbersteps_distriburion_on_rate.html" width=800 height=600 frameBorder=0></iframe>

  I have performed the permutation for 1000 times and then plot the distribution of simulated results about the absolute difference. (red vertical line is the observed statistics) 

<iframe src="n_steps_permutation_differences.html" width=800 height=600 frameBorder=0></iframe>

  The p-value I received is 0.0. As a 0.01 as a significance threshold is used and 0.01 > 0.0, we reject the null hypothesis that the rating is not dependent on the number of steps. We can directly observe from the plot as well. Therefore, we can conclude that the missingness of rating is the MAR because the rating is dependent on the number of steps. Probably if some recipes with relatively more or less number of steps will be more likely to have the missingness in the rating.

---
## Hypothesis Testing

Hypothesis Question: How nutritional value (calories, in particular) influences the user ratings of recipes

**Null Hypothesis (H0):** 
The calorie content of a recipe does not have a significant impact on its average user rating.

**Alternative Hypothesis (H1):**
The calorie content of a recipe has a significant impact on its average user rating.

**Test Statistic:** 
Given that we're comparing means of two groups (high-calorie vs. low-calorie recipes), a suitable test would be a two-sample t-test if the data follows a normal distribution and has equal variances. As previously mentioned, a recommended daily calorie for woman and man are 1800/2400 respectively, by considering this fact, I have chosen 2000 as the threshold. 
**1% Significance Level will be applied**

> Data Preparation:
- Categorize recipes into high-calorie and low-calorie groups based on a chosen threshold.
- Calculate the average rating for each recipe.

p-value: 0.002236625967153927 < 0.01

> Conclusion:
```
Given the p-value, we find sufficient evidence to reject the null hypothesis in favor of the alternative hypothesis at the 1% significance level. This implies that the calorie content of recipes does indeed have a significant impact on their average user ratings. However, it's important to note that while our analysis indicates a significant association, it does not establish causation. Other unmeasured factors might also contribute to this observed relationship.

Our findings are relevant for recipe creators and culinary platforms, indicating that users' rating behavior may be influenced by nutritional content, particularly calories. This insight can guide content curation and recipe development, potentially aligning offerings more closely with user preferences and dietary considerations.
```