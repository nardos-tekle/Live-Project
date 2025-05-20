# Tech Academy Live-Project

## Introduction
**Role:** Data Scientist

Google Play Store App Insights is a data-driven project I worked on as a Data Scientist through The Tech Academy, focusing on analyzing the Google Play Store dataset. The dataset included key app metrics such as installs, categories, prices, content ratings, genres, and update/version information. My work was managed using Microsoft Azure DevOps, and the project followed a Scrum-based methodology—complete with daily standups, sprint planning, and retrospectives to simulate a collaborative development environment.

Throughout the sprint, I completed four user stories that each focused on data cleaning, filtering, and visualization to help inform future decisions around app sales and strategy. I primarily used Python (Jupyter Notebook), RStudio, and Power BI to conduct my analysis and create visual insights.

In this report, I’ll briefly walk through the methodology, key findings, and tools used for each user story. I’ll also reflect on some of the challenges I encountered and what I learned from this rewarding experience.


## Core Technologies
* Juypter Notebook
* Rstudio
* PowerBI
* Packages: 
     - Python
        - Pandas, Numpy, Plotly, Seaborn, Matplotlib 
     - Rstudio
         - Dplyr, Tibble, Gt, Scales, Plotly, Tidyverse
* Dataset: [googleplaystore(col: 13 rows: 10842)](https://github.com/nardos-tekle/Live-Project/blob/main/Media/googleplaystoremaster.xlsx) 

## User Stories:
* [Content Rating](#user-story-1-content-rating)
* [Apps per Category](#user-story-2-apps-per-category)
* [Rating vs Installs](#user-story-3-ratings-vs-installs)
* [Data Analysis](#user-story-4-data-analysis)



*Jump To: [Introduction](#introduction), [Core Technology](core-technologies), [User Stories list](#user-stories), 
[User Story 2](#user-story-1-content-rating), [User Story 2](#user-story-2-apps-per-category), [User Story 3](#user-story-3-ratings-vs-installs), [User Story 4](#user-story-4-data-analysis), [Key Skills & Learning](#key-skills-learning)*



##

## User Story 1: Content Rating
**Request:** The client requested for the amount of Entertainment apps on Google Play Store that rate Mature. 

### Methodology
For this user story, I used R to analyse the datase as it does well with the filteration of huge datasets. 

I began cleaning the googleplay dataset by first renaming the "Content Rating" column to "Content" to keep things simple for the filtering process. Then started filtering the dataset to only include Entertainment apps and removed any duplicates using the **distinct()** function. 

```c#
# Renaming the content rating column to content for easier use
colnames(googleplay)[9] ="Content"

# Filter: Keep only entertainment apps from the full dataset
googleplay2 <- filter(googleplay, Category == "ENTERTAINMENT" ) %>%
  distinct()

# Select & Duplicate: Keep select columns and remove duplicates
contentdist<- subset(googleplay2, select = c('App', 'Content', 'Reviews', "Rating")) %>%
distinct() # Ensure results are unique
```
For a bounded version of the 'Reviews' column, I utilized the **mutate()** function within [dpylr](#core-technologies) by placing a capped dataset between 1,000,000 - 303, which are the max and min of this dataset, into a column review_capped. For readability, I then labeled the data based on the condition using the **comma()** from [scales](#core-technologies). 

``` c#
contentdist<- contentdist %>%
  mutate(
    Reviews_capped = ifelse(Reviews > 1000000, 1000000,
                            ifelse(Reviews < 303, 303, Reviews)),
    Reviews_label = case_when(
      Reviews > 1000000 ~ paste0(comma(1000000), "+"),
      Reviews < 303 ~ paste0(comma(303), "-"),
      TRUE ~ comma(Reviews)
    )
  )
```

Since the client requested for an total amount of apps within the Entertainment category. I then curated table that filtered the app count within each content rating. As well as compressed the review column of the dataset 

 ```c#
 # Group & Count: Count number of apps per content rating type
contentdist3<- contentdist %>%
  distinct()%>%
  filter(Content %in% c("Everyone", "Mature 17+", "Teen")) %>%
  group_by(Content) %>%
  summarise(Count = n(), .groups = "drop") 

```
I used an Interactive Barplot using [plotly](#core-technologies) showcasing the amount of apps per content category/rating. With user friendly hover tools displaying information of category and app content. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Barplotcontentapps-ezgif.com-video-to-gif-converter.gif" >
</p>
<br>

For more visual analysis, I compared the content ratings per each app with a interactive hover app that displays the apps rating, review, app name, and content rating. For aesthetic purposes, the bars are colored based on is respective rating categories

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/AppsbyCategoryR-ezgif.com-video-to-gif-converter.gif">
</p>


Finally, for a clearier display I created a visual table listing all Entertainment apps that are rated Mature. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Table%20of%20the%20apps.png">
</p>

### Findings and Conclusions

There is a total of six entertainment apps that are rated Mature. The average user rate for apps within the Mature category being 4.25. Mature has the smallest amount of apps making about a 5.5% proportion of apps within the entertainment category. 

Showcasing that there aren't as many entertainment apps rated for mature audiences, through these visualizations they might highlight the small number of apps that are and how they perform in terms of user feedback. 



*Jump To: [Introduction](#introduction), [Core Technology](core-technologies), [User Stories list](#user-stories), 
[User Story 2](#user-story-1-content-rating), [User Story 2](#user-story-2-apps-per-category), [User Story 3](#user-story-3-ratings-vs-installs), [User Story 4](#user-story-4-data-analysis), [Key Skills & Learning](#key-skills-learning)*






## User Story 2: Apps per Category
**Request:** The client requested for the amount of apps in certain categories from the following categories: 
  - Entertainment
  - Social
  - Productivity
    
**Additional Tasks:**
- The client would like to know how many of the apps in each category are free or paid.
- The client would like to know which app in each category is the highest priced and which app is the lowest priced.
- The client would like a list of the top 5 highest priced apps and their categories.

### Methodology
For this story I utilized Python's Juypter Notebook for it's live coding ability, that comes in handy for data the visuals for the tasks. 

I started cleaning the dataset by removing NA values and duplicated entries based on the "App" column to showcase only unqiue values. 

```c#
googleplay_df2['App'].value_counts() #the amount of times an app name was listed
googleplay_df2 = googleplay_df2.dropna() # drops na values within the dataset

googleplay_df2.loc[googleplay_df2.duplicated(subset=['App'], keep=False)] #Removes the duplicated rows within the app column
```
However, I was met with some difficulty as when I checked the .valuecounts() in the app column each app still displayed multiple outputs. When investigating, the the data within the rows needed to be filtered. To do this, I used the **.sort_values()** from [pandas](#core-technologies) to sort the data within the reviews column from biggest to smallest. Then removed the duplicated rows by keeping the rows that come first that are now sorted. 

```c#
googpleplay_df2 = googleplay_df2.sort_values(by='Reviews', ascending=False) # sorts the values within the review column from biggest to smallest
googleplay_df2 = googleplay_df2.drop_duplicates(subset='App', keep='first') # removes other duplicated rows and keeps the rows that come first now that it has been sorted. 
googleplay_df2['App'].value_counts()
```

The dataset was then filtered to only represent the requested categories. For the first analysis, the total amount of apps by category was visualized using [plotly's](#core-technologies) interactive pie chart with color aesthetics to indicate the categories and hover features with data information. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/totall%20apps%20by%20category.gif">
</p>

For the next analysis of the highest and lowest apps per category, I grouped the data within the 'Category' column by using **.agg()** method from [pandas](#core-technologies) to aggregate the maximum and minimum prices into new columns, max_price and min_price . Then retrieve the highest and lowest priced apps in each category by labeling "Highest" and "Lowest"  into a new column, price_type. 

```c#
# assigns variable to group the categories max and min prices and set them as values
category_prices = filtered_df.groupby('Category')['Price'].agg(Max_Price='max', Min_Price='min').reset_index()

# Assigns variable to showcase the highest priced apps
highest_apps = filtered_df.loc[filtered_df.groupby('Category')['Price'].idxmax()]
highest_apps['Price_Type'] = 'Highest' # labels the value the highest in the price_type column

# Assigns variable to showcase the lowest priced apps
lowest_apps = filtered_df.loc[filtered_df.groupby('Category')['Price'].idxmin()]
lowest_apps['Price_Type'] = 'Lowest' # labels them the lowest within the price_type column

# Combines both of them into one dataframe
filtered_df_high_low = pd.concat([highest_apps, lowest_apps], ignore_index=True)
print(filtered_df_high_low)
```
For visualization, I created an interactive grouped barplot analyzing the app categories by price and visual aesthetics for their respective price_type and hovers for detailed information on the app's name, price, category, and price_type. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Highest%3ALowest%20priced%20apps.gif">
</p>

For the last analysis, I used a tooltip scatterplot representing the top 5 highest priced by category. Using the app name for the visual aesthetics and hovers for detailed information. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Top5expensiveapps.gif">
</p>

### Findings and Conclusion
The app store landscape shows clear trends across Entertainment, Social, and Productivity categories. Productivity leads in app count (301), followed by Social (203) and Entertainment (102). The vast majority of apps are free (584), with only 22 being paid, indicating a strong preference for free content among users. Interestingly, the Social category had only one paid app, priced modestly at $0.99.

Pricing insights show that Productivity apps dominate the high-end market, with the top five most expensive apps all falling within that category—topping out at $8.99. Entertainment apps fall in the mid-range ($2.99–$4.99), while Social apps have the narrowest pricing. These findings highlight where monetization opportunities lie and can guide future app development and marketing strategies



*Jump To: [Introduction](#introduction), [Core Technology](core-technologies), [User Stories list](#user-stories), 
[User Story 2](#user-story-1-content-rating), [User Story 2](#user-story-2-apps-per-category), [User Story 3](#user-story-3-ratings-vs-installs), [User Story 4](#user-story-4-data-analysis), [Key Skills & Learning](#key-skills-learning)*







## User Story 3: Ratings vs Installs
**Request:** The client would like to know if there is a correlation between content rating and installs 

### Methodology

**Columns/Data to Analyze:**
* Category
* Content Rating 
* Installs

* Look over each content rating and number of installs.

* Determine which content rating has the highest number of installs and how many categories it belongs to. 


For this story, utilizing the filtering method's from the previous user story. Once, I retrieved the neccessary columns for proper analysis. Due to wide set of data within the 'Installs' for cleaner representation. I categorized the data represent 1,000,000,000 to 1B+, numbers greater than or equal to 100,000 to 100K-1B, and <100K for numbers that are any less. 

```c#
def categorize_installs(installs): # Sets categories for the installs for cleaner representation of the huge dataset
    if installs >= 1_000_000_000:
        return "1B+"
    elif installs >= 100_000:
        return "100K–1B"
    else:
        return "<100K"

filtered_set1['Install Group'] = filtered_set1['Installs'].apply(categorize_installs) # creates a new columnn and places into the dataset. 
```
For bigger picture for analysis of the number installs per each category, I used the **.groupby()** from [pandas](#core-technologies) to group the following columns and sums the 'Installs' per each category making it tidy into a new column. 

```c#
content_installs = filtered_set1.groupby(['Content Rating', 'Category', 'Install Group'])['Installs'].sum().reset_index(name='Total Installs') # Groups the content rating and category columns and total ups the installs within those two rows.
print(content_installs)

filtered_installs = content_installs.groupby(['Content Rating', 'Install Group'])['Total Installs'].sum().reset_index() #Groups up all of the content ratings and totals the amount of installs per each rating
print(filtered_installs)
```
To visual this, I created interactive donut pie chart showing the total installs by each content rating. Color aesthetics showcasing the content ratings and hover effects for further information. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Total%20installs%20by%20content%20rating.gif">
</p>

I then grouped the content ratings and aggregated the total installs and unique categories per each rating. For visualization, Using an Interactive Barplot showcasing these variables with hover and aesthestics for visual analysis and futher information 

```c#
# Grouping the content ratings and aggregating the total installs per each rating as well as the unique categories that make up the two rows.
installs_summary = content_installs.groupby('Content Rating').agg({
    'Total Installs': 'sum',
    'Category': pd.Series.nunique
}).reset_index()

# Filtering the three columns. 
installs_summary.columns = ['Content Rating', 'Total Installs', 'Unique Categories']
print(installs_summary)

# interactive barplot showing the total installs per each content rating and how many respective categories per each. 
categoryfig = px.bar(installs_summary, x="Content Rating", y="Total Installs", color="Unique Categories", title="Total Installs and Categories by Content Ratings")
categoryfig.show()
```

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Totall%20installs%3ACategories%20by%20%20Rating.gif">
</p>

For the next analysis, I grouped the content rating by the highest installs. Using an interactive tooltip scatterplot to showcase the top highest total installs by category with visual aesthetics to showcase the categories. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Totall%20installs%20by%20category%20and%20content%20rating.gif">
</p>


Lastly, to better visualize the distribution of installs by categories and content rating. I created interactive stacked barchart displaying the distribution of installs per each category with visual aesthetics showcasing its respective content ratings. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Highest%20totall%20installs%20by%20category.gif">
</p>

### Findings and Conclusion

The data reveals a clear correlation between content rating and app installs. Apps rated “Everyone” have the highest install volume (~50B) and appear in the most categories (33), suggesting that broader accessibility directly contributes to greater reach. Similarly, “Teen” and “Everyone 10+” also perform well, though at lower scales, while age-restricted ratings like “Mature 17+” and “Adults only 18+” see far fewer installs and are present in fewer categories.

This suggests that the more inclusive the content rating, the higher the potential for downloads and category coverage. For the client, this means prioritizing “Everyone” or similarly broad ratings could help maximize visibility and user engagement across the app market.



*Jump To: [Introduction](#introduction), [Core Technology](core-technologies), [User Stories list](#user-stories), 
[User Story 2](#user-story-1-content-rating), [User Story 2](#user-story-2-apps-per-category), [User Story 3](#user-story-3-ratings-vs-installs), [User Story 4](#user-story-4-data-analysis), [Key Skills & Learning](#key-skills-learning)*







## User Story 4: Data Analysis
**Request:** The client wants an analysis to determine which app categories are most likely to have high ratings and installs, and how these are influenced by pricing, in-app purchases, and other factors

### Methodology

**Analysis:**
* Which app categories tend to receive the highest ratings and the most installs?

* How does pricing and in-app purchasing options impact the ratings and installs of apps across different categories?

* Are there any trends over time in which certain app categories gain popularity, based on installs?

To prepare the dataset for analysis and visualization, several cleaning steps were performed:

I started to clean up the dataset by dropping missing values Dropping missing values in the `Rating` column using the **.dropna()** method to ensure accuracy in averages. Then converted columns from a string (e.g., "1,000+") to an integer by removing non-numeric characters and converting to numeric for `Installs` and float for `Price` . 

```c#

# Drop rows where Rating is NaN
df = df.dropna(subset=['Rating'])

# 1. Ensure Installs is treated as string
df['Installs'] = df['Installs'].astype(str).copy()

# 2. Keep only rows with valid patterns like '1,000+' (no 'Free', 'NaN', etc.)
df = df[df['Installs'].str.contains(r'^\d+[+,]*$', na=False)]

# Now clean and convert
df['Installs'] = df['Installs'].str.replace('[+,]', '', regex=True).astype(int)

# Same with Price — force string, clean, convert
df['Price'] = df['Price'].astype(str)

# Strip dollar sign in Price and convert to float
df['Price'] = df['Price'].replace(r'[\$,]', '', regex=True).astype(float)

# Dropping missing values in Rating Column


df = df.dropna(subset=['App', 'Category'])

# Impute or drop remaining missing values
df['Rating'] = df['Rating'].fillna(df['Rating'].median())
df['Installs'] = df['Installs'].fillna(df['Installs'].median())
```
Removed outliers within the `Rating` and `Installs`  

Confirmed app types using the existing `Type` column (`Free` vs `Paid`). 

```c#
df['Type'] = df['Type'].str.strip().str.title()  # Ensures values are like 'Free' or 'Paid'

# Compare average installs and ratings for free vs. paid apps
price_rate = df.groupby('Type').agg({'Rating': 'mean', 'Installs': 'mean'})
print(price_rate)

# Compare by Category and Type
category_price = df.groupby(['Category', 'Type']).agg({'Rating': 'mean' , 'Installs': 'mean'}).unstack()
print(category_price)
```
Lastly, to find trends over time the column `Last Updated` column was converted to a proper datetime format to showcase the year. Then group the data by the `Year`, `Category`, `Installs` for each row. The cleaned data was saved to a new CSV file and imported into Power BI for visualization.


```c#
df['Type'] = df['Type'].str.strip().str.title()  # Ensures values are like 'Free' or 'Paid'

# Compare average installs and ratings for free vs. paid apps
price_rate = df.groupby('Type').agg({'Rating': 'mean', 'Installs': 'mean'})
print(price_rate)

# Compare by Category and Type
category_price = df.groupby(['Category', 'Type']).agg({'Rating': 'mean' , 'Installs': 'mean'}).unstack()
print(category_price)

# Convert to datetime if it exists
df['Last Updated'] = pd.to_datetime(df['Last Updated'], errors='coerce')

# Extract year or month
df['Year'] = df['Last Updated'].dt.year

# Group by year and category
trend_time = df.groupby(['Year', 'Category'])['Installs'].sum().unstack
print(trend_time)

# Filtered Datasets turned into CSV files
price_rate.to_csv("price_rate.csv")
category_price.to_csv("category_price.csv")
top_rated_categories.to_csv("toprated_categories.csv")
top_installed_categories.to_csv("topinstalled_categories.csv")
trend_time.to_csv("trend_time.csv")
```

**Power BI Visualizations:**

For the analysis on `Installs` and `Ratings` against `Price Type` I created a ribbon barplot. I chose this because of its nice feature in visualizing the distribution between the two variables. As well as, listing the difference in Ratings and Installs for in apps that are Free or Paid.


<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Installs%20and%20ratings%20by%20price.gif">
</p>


I created a funnel bar plot for the Rating by Category analysis. I chose this visualization because it clearly highlights the distribution of app ratings across different categories. Although we didn’t fully utilize the conversion rate feature in this context, the funnel chart still provides a strong visual structure. It gives us the potential to start incorporating conversion-related insights in future analyses, making it a valuable tool for visualizing progression or comparisons across stages or groups.

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Rating%20by%20category.gif">
</p>


For the next analysis on Installs and Ratings by Category, I created a pie chart to showcase a bigger picture of the proportions of which categories had the most and least installs/ratings overall. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Installs%20and%20Rating%20by%20category.gif">
</p>

For Installs by Category a donut chart was used to showcase once again a porportion based analysis of the total installs per category. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/installs%20by%20category.gif">
</p>

Final analysis on Categorical trends by the years, I created a line chart to visualize the amount of installs per category from the years of 2011-2018. 

<p align=center>
<img src="https://github.com/nardos-tekle/Live-Project/blob/main/Media/Time-Trend.gif">
</p>



*Jump To: [Introduction](#introduction), [Core Technology](core-technologies), [User Stories list](#user-stories), 
[User Story 2](#user-story-1-content-rating), [User Story 2](#user-story-2-apps-per-category), [User Story 3](#user-story-3-ratings-vs-installs), [User Story 4](#user-story-4-data-analysis), [Key Skills & Learning](#key-skills-learning)*







## Key Skills & Lessons Learned

### Adapted to Independent Scrum Development:

Throughout the sprint, I followed Scrum methodology using Microsoft Azure DevOps. Although I worked independently, I practiced structured project management with sprint planning, daily stand-ups, and retrospectives. This helped me stay organized, meet deadlines, and approach each user story with clarity and focus.

### Data Analysis & Visualization:

I used Python (Jupyter Notebook), RStudio, and Power BI to conduct analysis and create visualizations based on Google Play Store app data. This involved cleaning and filtering data across variables such as category, price, content rating, and installs. Every user story I worked on incorporated data visualization as a core deliverable to support potential product strategy or future app development.

### Problem-Solving & Debugging:

One of the most challenging aspects was debugging, especially since I was still getting comfortable with some of the languages and tools. I leaned heavily on independent research, AI tools, and occasionally reached out to my mentors for support. At one point, I accidentally deleted a large portion of the dataset and had to start over from scratch. That experience taught me a valuable lesson: always use .copy() when manipulating data, and consider working in branches or backups when making significant changes.

### Growth in Analytical Thinking:

This project really expanded my thinking around how to structure and grow my analysis. I began to better understand how to break down raw data into actionable insights and how different visual tools could bring clarity to patterns that might otherwise go unnoticed.

### Real-World Application:

Even though the sprint was short, the hands-on nature of the work gave me a realistic glimpse into what it's like to handle and analyze messy, real-world data. The lessons I’ve learned—both technical and practical—will definitely carry over into future projects and data-related roles.

*Jump To: [Introduction](#introduction), [Core Technology](core-technologies), [User Stories list](#user-stories), 
[User Story 2](#user-story-1-content-rating), [User Story 2](#user-story-2-apps-per-category), [User Story 3](#user-story-3-ratings-vs-installs), [User Story 4](#user-story-4-data-analysis), [Key Skills & Learning](#key-skills-learning)*
