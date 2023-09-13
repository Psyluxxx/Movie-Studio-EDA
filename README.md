# IMDb Movie Data EDA

**Authors**: Franko Ndou, Anthony Brocco

## Overview

IMDb has provided access to an SQL database containing extensive movie data, in addition to two CSV files we obtained. These resources facilitate an Exploratory Data Analysis (EDA) aimed at addressing complex business challenges. Our primary goal is to identify the top-performing films in the current box office and translate our findings into comprehensible data visualizations and recommendations.

## Business Problem

Universal Pictures aspires to produce the next blockbuster film with a substantial budget. Their vision includes assembling the finest directors, actors, and implementing optimal business strategies to create not only a generation-defining film but also to maximize return on investment (ROI). Our task is to conduct an in-depth Exploratory Data Analysis (EDA) using extensive datasets to help Universal reach a conclusion on the most effective approach to achieving this ambitious goal.

## Data

We are using multiple data sets, mainly the IMDb database as well as two other CSV files that can all be found [here](https://github.com/learn-co-curriculum/dsc-phase-2-project-v3/tree/main/zippedData)

## Creating the Production Team

Assembling a top-notch production team is essential for creating a successful film. Identifying the best director and writer for the job is crucial. While actors play significant roles, directors often craft roles with specific actors in mind. Therefore, determining the most successful actor may not directly contribute to our production team's ability to make the best possible movie. The success of a film largely hinges on the artistic vision of the director and the script quality. Relying solely on statistics related to actors may not enhance our return on investment (ROI) and could potentially have a detrimental impact on the film's quality.

## Setting Up the Workspace

```python
# Importing libraries
import pandas as pd
import sqlite3 
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats
import numpy as np
import warnings

# Ignores warnings 
warnings.filterwarnings("ignore")

# Creating data frames and establishing connections
budgets = pd.read_csv('../zippedData/movie_budget_cleaned.csv')
gross = pd.read_csv('../zippedData/gross_movie_cleaned.csv')
conn = sqlite3.connect('../zippedData/im.db')
```

## Hypothesis Testing

Statistical analysis is used to assess the likelihood and confidence of Christopher Nolan's movies outperforming those of other directors. This analysis provides valuable insights into the comparative performance of different directors.

```python
# Performing Nolan test
stats.ttest_1samp(c_nolan_films['ROI'], 1.74)

# Performing Fincher test
stats.ttest_1samp(d_fincher_films['ROI'], 1.74)

# Performing Abrams test
stats.ttest_1samp(j_abrams_films['ROI'], 1.74)
```

Based on the sample set we have, there is a 3.8% chance that Christopher Nolan's films will not achieve an ROI of 1.74 or higher. This suggests a high likelihood (96.2%) that Christopher Nolan will indeed generate a return on investment for us. Consequently, employing Christopher Nolan as the director and potentially as the writer for our film is strongly recommended, aligning with the data-driven evidence.

## Which Genres Have the Largest ROI?

Action movies have the highest ROI among different genres by a significant margin. This aligns perfectly with the goal of working with a director who excels in this genre.

```python
# Create a bar plot for average ROI by genre
plt.figure(figsize=(12, 6))
sns.barplot(data=genre_roi_avg, x='genre', y='ROI', palette='viridis')
plt.xticks(rotation=90)
plt.xlabel('Genre')
plt.ylabel('Average ROI')
plt.title('Average ROI by Genre')
plt.tight_layout()
plt.show()
```

## Best Time to Release Films

Wednesday is the best day to release our movie, with the highest ROI. Releasing during the warmer months, such as June, July, and May, is recommended as people are more likely to go to the movies during this period.

```python
# Create bars for day of the week vs. domestic gross
sns.barplot(x='day_of_the_week', y='domestic_gross_in_mill', data=budgets)
sns.set(font_scale=0.75)
plt.ylabel('Domestic Gross per million')
plt.title('Domestic Gross on Given Day of the Week')
plt.show()

# Create bars for month of the year vs. domestic gross
sns.barplot(x='month_of_the_year', y='domestic_gross_in_mill', data=budgets)
sns.set(font_scale=0.75)
plt.xticks(rotation=45)
plt.ylabel('Domestic Gross per million')
plt.title('Domestic Gross by Month of the Year')
plt.show()
```

## Finding the Right Musician for the Score

Selecting a talented musician with a track record of working on successful films is crucial for creating a captivating soundtrack that enhances the overall impact and success of our movie.

```python
# Merge two DataFrames 'merged_df' and 'people_and_movies_df' using an outer join
relevant_people_and_movies = pd.merge(merged_df, people_and_movies_df, how='outer', left_on='movie', right_on='original_title')

# Drop columns we don't need
relevant_people_and_movies = relevant_people_and_movies.drop(['release_date',
  'runtime_minutes', 'original_title', 'genre'], axis=1)

# Sort the DataFrame by 'ROI' in descending order
relevant_people_and_movies = relevant_people_and_movies.sort_values(by='ROI', ascending=False)

# Drop rows with missing 'primary_profession' values
relevant_people_and_movies = relevant_people_and_movies.dropna(subset=['primary_profession'])

# Filter the DataFrame based on primary profession
chosen_artists = relevant_people_and_movies[relevant_people_and_movies['primary_profession'].str.contains \
                                                  ('soundtrack|composer|music_department|sound_deparment')]
# Select relevant columns
chosen_artists = chosen_artists[['primary_name', 'primary_profession', 'ROI', 'averagerating', 'numvotes']]

# Drop duplicate rows, if any
chosen_artists = chosen_artists.drop_duplicates()

# Sort by 'average rating' in descending order 
chosen_artists = chosen_artists.sort_values(by='averagerating', ascending=False)

# Chose only 'successful' artists by setting the roi to 2 as well as the minimum rating to 7 and display final result
chosen_artists = chosen_artists[chosen_artists['ROI'] > 1.74]
chosen_artists = chosen_artists[chosen_artists['numvotes'] > 10000]
chosen_artists[chosen_artists['averagerating'] >= 7]
```

## Backup Directors

We've identified potential backup directors in case our preferred choice is unavailable.

```python
# Merge two DataFrames 'merged_df' and 'people_and_movies_df' using an outer join
relevant_people_and_movies = pd.merge(merged_df, people_and_movies_df, how='outer', left_on='movie', right_on='original_title')

# Drop columns we don't need
relevant_people_and_movies = relevant_people_and_movies.drop(['release_date',
  'runtime_minutes', 'original_title', 'genre'], axis=1)

# Sort the DataFrame by 'averagerating' in descending order (changed from 'ROI')
relevant_people_and_movies = relevant_people_and_movies.sort_values(by='averagerating', ascending=False)

# Drop rows with missing 'primary_profession' values
relevant_people_and_movies = relevant_people_and_movies.dropna(subset=['primary_profession'])

# Filter the DataFrame based on primary profession (director)
chosen_directors = relevant_people_and_movies[relevant_people_and_movies['primary_profession'].str.contains('director')]

# Select relevant columns
chosen_directors = chosen_directors[['primary_name', 'primary_profession', 'averagerating', 'numvotes']]

# Drop duplicate rows, if any
chosen_directors = chosen_directors.drop_duplicates()

# Sort by 'averagerating' in descending order (optional)
chosen_directors = chosen_directors.sort_values(by='averagerating', ascending=False)

# Filtering for above 7 rating
chosen_directors_filtered = chosen_directors[chosen_directors['averagerating'] >= 7.0]

# Display the directors with the highest ratings
chosen_directors_filtered.dropna().reset_index()
```

## Conclusion

This EDA provides valuable insights into making data-driven decisions for Universal Pictures. It highlights key recommendations, such as producing an action movie, releasing on Wednesdays, working with Christopher Nolan, and selecting talented musicians. While data analysis cannot guarantee success, it enhances the likelihood of achieving the desired ROI and critical acclaim. Additional steps like predictive modeling, A/B testing, and market analysis can further refine decision-making.

## Next Steps

To provide even more insight for Universal Pictures, these are the steps we could take

- Predictive Modeling: We can build predictive models using machine learning techniques. We could use features like director, genre, release date, and budget to predict box office performance. This would require splitting our data into training and testing sets, selecting appropriate algorithms, and evaluating model performance.

- A/B Testing: We could consider running A/B tests for factors like release date or marketing strategies to determine their impact on box office success. A well-designed A/B test can provide valuable insights.

- Market Analysis: We could conduct a deeper market analysis to understand the competitive landscape and audience preferences. Analyze trends in the movie industry, such as the rise of streaming services and changing consumer behavior.

- Budget Optimization: Explore cost optimization strategies, such as how to allocate the budget effectively across different aspects of movie production, including marketing and distribution.

## For More Information

Please review our full analysis in [our Jupyter Notebook](./code/Analysis_of_Movie_Data.ipynb) or our [presentation](./Movie_EDA_Presentation.pdf).

For any additional questions, please contact us:

**Franko Ndou & frankondou@gmail.com, Anthony Brocco & anthonybrocco98@gmail.com**

## Repository Structure

Describe the structure of your repository and its contents, for example:

```
├── code
│   ├── __init__.py
│   ├── Cleaning-Movie-Data.ipynb
│   └── Analysis_of_Movie_Data.ipynb
├── images
├── README.md
├── .gitignore
└── Movie_EDA_Presentation.pdf
```
