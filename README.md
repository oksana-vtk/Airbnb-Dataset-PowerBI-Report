
This repository contains my analysis of the **Airbnb Listings & Reviews** dataset, 
along with calculated measures and an interactive Power BI Report.

📁 Dataset available [here](https://www.kaggle.com/datasets/mysarahmadbhat/airbnb-listings-reviews/data)

📊 My Power BI Report available [here](https://app.powerbi.com/view?r=eyJrIjoiMTI4NjQxYWItZmQwMS00OGNhLTgxODgtOGMwNzExYjk3MzFlIiwidCI6ImIwYmYzYTRlLTBlMmMtNGQ5Ny1hMzUyLWY2MDY4MGFkYjZlMSIsImMiOjl9)

******
## Project: Airbnb Listings & Reviews
Airbnb data for 250,000+ listings across 10 major cities, with 5 milion reviews.

### About Dataset
Airbnb data for 250,000+ listings in 10 major cities, including information about hosts, pricing, location, and room type, along with over 5 million historical reviews.
NOTE: Prices are in local currency

### Recommended Analysis
- Can you spot any major differences in the Airbnb market between cities?
- Which attributes have the biggest influence in price?
- Are you able to identify any trends or seasonality in the review data?
- Which city offers a better value for travel?

****

## My Data Analysis Steps in Power Query
* First, I analyzed the data in the fact tables `Listings` and `Reviews`, performing data cleaning 
and transformation.
* I explored the `amenities` column in the `Listings` table, split the list of amenities for each listing_id into 
individual values, and standardized them. As a result, I created a new fact table `Listing-Amenities` 
with the columns listing_id and amenity_cleaned.
* I created the following dimension tables: `DimHost`, `DimCity`, `DimRoomType`, and `DimAmenities`.
* Since the price in the `Listings` table is in local currency,  I collected historical exchange rates to USD for each country in the dataset.
These were sourced from the official websites of national banks or financial institutions and cover the full observation period from 2008 to 2021.

Bangkok - Thailand - THB - Thai Baht
Cape Town - South Africa - ZAR - South African Rand
Hong Kong - Hong Kong - HKD - Hong Kong Dollar
Istanbul - Turkey - TRY - Turkish Lira
Mexico City - Mexico - MXN - Mexican Peso
New York - USA - USD - American Dollar
Paris - France - EUR - Euro
Rio de Janeiro - Brazil - BRL - Brazilian Real
Rome - Italy - EUR - Euro
Sydney - Australia - AUD - Australian Dollar

* Based on this data, I created a fact table `Exchange Rates`.
* I also created a `Calendar` Date table:

<pre>Calendar = CALENDAR(MIN(Listings[host_since]),MAX(Reviews[date]))</pre>

## My Key Metrics Calculation in DAX

<details>
  <summary><strong>Power BI Report, Page 1</strong></summary>

  🌟 Total revenue:
  <pre>Total_revenue = SUM(Listings[Price_in_usd])</pre>

  🌟 Total average price:
  <pre>total_price_avg = CALCULATE([price_usd_avg], ALL(Listings))</pre>

  🌟 Most visited city:
  <pre>Most_visited_city = 
  VAR TopCityTable = TOPN(1, ALL(DimCity), [Accomodates_total], DESC)  
  RETURN 
  CONCATENATEX(TopCityTable, DimCity[city], ", "))</pre>
  
  🌟 Quantity of new listings:
  <pre>New_Listing Qty = COUNT(Listings[listing_id])</pre>
  
  🌟 Quantity of new hosts:
  <pre>New_Host Quantity = DISTINCTCOUNT(Listings[host_id])</pre>
  
  🌟 New Listings Quantity depends on selected year (for title):
  <pre>New_Listing Qty for title = 
   VAR user_year = SELECTEDVALUE('Calendar'[Year])
   VAR last_year_list_qty = CALCULATE(COUNT(Listings[listing_id]), FILTER('Calendar', 'Calendar'[Year] = 2021))
   RETURN
   IF(user_year = BLANK(), last_year_list_qty, [New_Listing Qty])</pre>
  
  🌟 New Host Quantity depends on selected year (for title):
  <pre>New_Host Qty for title = 
   VAR user_year = SELECTEDVALUE('Calendar'[Year])
   VAR last_year_host_qty = CALCULATE(DISTINCTCOUNT(Listings[host_id]), FILTER('Calendar', 'Calendar'[Year] = 2021))
   RETURN
   IF(user_year = BLANK(), last_year_host_qty, [New_Host Quantity])</pre>
  
  🌟 Total listings quantity as a comulative quantity:
  <pre>Comul_listing_qty = 
  CALCULATE([New_Listing Qty],FILTER(ALL('Calendar'[Date]), 'Calendar'[Date]<=MAX('Calendar'[Date])))</pre>
  
  🌟 Total hosts quantity as a comulative quantity:
  <pre>Comul_host_qty =
  CALCULATE([New_Host Quantity],FILTER(ALL('Calendar'[Date]), 'Calendar'[Date]<=MAX('Calendar'[Date])))</pre>
  
  🌟 Total guests:
  <pre>Accomodates_total = SUM(Listings[accommodates])</pre>
  
  🌟 Quantity of old (already existed) listings for the selected year:
  <pre>Old_comul_list_qty = [Comul_listing_qty] - [New_Listing Qty]</pre>
  
  🌟 Average price:
  <pre>price_usd_avg = DIVIDE([price_usd],[New_Listing Qty])</pre>

</details>

<details>
  <summary><strong>Power BI Report, Page 2</strong></summary>

  🌟 Quantity of hosts who have a profile picture:
  <pre>Host_profile_pic_qty = 
  CALCULATE([Comul_host_qty], FILTER(DimHost, DimHost[host_has_profile_pic]="t"))</pre>
  
  🌟 Quantity of hosts who have a verified id:
  <pre>Host_id_verified_qty = 
  CALCULATE([Comul_host_qty], FILTER(DimHost, DimHost[host_identity_verified]="t"))</pre>
  
  🌟 Quantity of hosts who are a superhost:
  <pre>Host_is_superhost_qty = 
  CALCULATE([Comul_host_qty], FILTER(DimHost, DimHost[host_is_superhost]="t"))</pre>
  
  🌟 Quantity of hosts filtered by Response time (withing an hour, withing a few hours, withing a few days, a few days or more):
  <pre>Response_time_host_qty = 
  VAR response_time = SELECTEDVALUE('DimHost'[host_response_time])
  RETURN
  IF(ISBLANK(response_time), CALCULATE([New_Host Quantity], ALL(DimHost)),
  CALCULATE([New_Host Quantity],FILTER(DimHost, DimHost[host_response_time] = response_time)))</pre>
  
  🌟 Host Response Rate by Rates or Cities:
  <pre>Response_rate_host_qty = 
  VAR user_rate = SELECTEDVALUE('Rate'[rates])
  RETURN
  IF(ISBLANK(user_rate), CALCULATE([New_Host Quantity], ALL(DimHost)),
  CALCULATE([New_Host Quantity],
  FILTER(DimHost, DimHost[host_response_rate] <= VALUE(user_rate) && DimHost[host_response_rate] > VALUE(user_rate - 0.1))))</pre>
  
  🌟 Host Response Rate by Rates or Cities:
  <pre>Response_rate_host_qty_optim = 
  VAR user_rate = SELECTEDVALUE('Rate'[rates])
  VAR lower_bound = VALUE(user_rate - 0.1)
  RETURN
  IF(
      ISBLANK(user_rate),
      CALCULATE([New_Host Quantity], ALL(DimHost)),
      CALCULATE(
          [New_Host Quantity],
          FILTER(
              DimHost,
              DimHost[host_response_rate] <= user_rate &&
              DimHost[host_response_rate] > lower_bound
          )
      )
  )</pre>
  
  🌟 Hosts Acceptance Rate by Rates or Cities:
  <pre>Acceptance_rate_host_qty = 
  VAR user_rate = SELECTEDVALUE('Rate'[rates])
  RETURN
  IF(ISBLANK(user_rate), CALCULATE([New_Host Quantity], ALL(DimHost)),
  CALCULATE([New_Host Quantity],
      FILTER(DimHost, DimHost[host_acceptance_rate] <= VALUE(user_rate) && 
      DimHost[host_acceptance_rate] > VALUE(user_rate - 0.1))))</pre>
  
  🌟 Scores Accuracy (for Ratings):
  <pre>Scores_accuracy = 
  DIVIDE(SUM(Listings[review_scores_accuracy]), CALCULATE([New_Listing Qty], FILTER(Listings, Listings[review_scores_accuracy] > 0)))</pre>
  
  🌟 Scores Checkin (for Ratings):
  <pre>Scores_checkin = 
  DIVIDE(SUM(Listings[review_scores_checkin]), CALCULATE([New_Listing Qty], FILTER(Listings, Listings[review_scores_checkin] > 0)))</pre>
  
  🌟 Scores Cleanliness (for Ratings):
  <pre>Scores_cleanliness = DIVIDE(SUM(Listings[review_scores_cleanliness]), CALCULATE([New_Listing Qty], FILTER(Listings, Listings[review_scores_cleanliness] > 0)))</pre>
  
  🌟 Scores Communication (for Ratings):
  <pre>Scores_communication = DIVIDE(SUM(Listings[review_scores_communication]), CALCULATE([New_Listing Qty], FILTER(Listings, Listings[review_scores_communication] > 0)))</pre>
  
  🌟 Scores Location (for Ratings):
  <pre>Scores_location = DIVIDE(SUM(Listings[review_scores_location]), CALCULATE([New_Listing Qty], FILTER(Listings, Listings[review_scores_location] > 0)))</pre>
  
  🌟 Scores Rating (for Ratings):
  <pre>Scores_rating = DIVIDE(SUM(Listings[review_scores_rating]), CALCULATE([New_Listing Qty], FILTER(Listings, Listings[review_scores_rating] > 0)))</pre>

</details>

<details>
  <summary><strong>Power BI Report, Page 3</strong></summary>

  🌟 Max listing's price:
  <pre>MAX_price = MAX(Listings[Price_in_usd])</pre>
  
  🌟 City with max listing's price:
  <pre>MAX_price_city = 
  VAR maxprice = MAX(Listings[Price_in_usd])
  RETURN 
  "City: " & CALCULATE(MAX(DimCity[city]), FILTER(Listings, Listings[Price_in_usd] = maxprice))</pre>
  
  🌟 Listing_id with max listing's price:
  <pre>MAX_price_listing_id = 
  VAR maxprice = MAX(Listings[Price_in_usd])
  RETURN 
  "Listing_id: " & CALCULATE(MAX(Listings[listing_id]), FILTER(Listings, Listings[Price_in_usd] = maxprice))</pre>
  
  🌟 Year with max listing's price:
  <pre>MAX_price_year = 
  VAR maxprice = MAX(Listings[Price_in_usd])
  RETURN 
  "Year: " & CALCULATE(MAX('Calendar'[Year]), FILTER(Listings, Listings[Price_in_usd] = maxprice))</pre>
  
  🌟 Dynamic TOP Amenities:
  
  - Created a calculated table:
      <pre>TOP = GENERATESERIES(10, 50, 5)</pre>
    - Created a slicer with a field `TOP[Value]`
  - <pre>Listing_amenit = COUNT('Listings-Amenities'[listing_id])</pre>
  - <pre>Listing_amenit % = [Listing_amenit] / [New_Listing Qty]</pre>
  - <pre>amenit_rank = RANKX(ALL(DimAmenities[Amenity]), [Listing_amenit],,DESC,Dense)</pre>
  - <pre>Listing_amenit_%_param = 
    VAR param = SELECTEDVALUE('TOP'[Value]) 
    RETURN IF([amenit_rank] <= param, [Listing_amenit %], BLANK())</pre>
  - <pre>Listing_amenit_param = 
    VAR param = SELECTEDVALUE('TOP'[Value])
    RETURN 
    IF([amenit_rank] <= param, [Listing_amenit], BLANK())</pre>
  - <pre>price_avg_by_amenities =
    VAR amenit = SELECTEDVALUE(DimAmenities[Amenity])
    VAR total_price_by_amenit = CALCULATE(SUM(Listings[Price_in_usd]),FILTER('Listings-Amenities','Listings-Amenities'[amenity_cleaned]=amenit))
    VAR total_list_by_amenit = CALCULATE(COUNT(Listings[Price_in_usd]),FILTER('Listings-Amenities','Listings-Amenities'[amenity_cleaned]=amenit))
    VAR avg_price = DIVIDE(total_price_by_amenit, total_list_by_amenit)
    VAR param = SELECTEDVALUE('TOP'[Value])
    RETURN
    IF([amenit_rank] <= param, avg_price, BLANK())</pre>
  
  🌟 Dynamic visual titles depends on selected year and/or selected city :
  <pre>Amenity_type_avg_price = 
  IF(
      ISBLANK(SELECTEDVALUE(DimCity[city])) && ISBLANK(SELECTEDVALUE('Calendar'[Year])), "Amenity Type and Amenities Average price", 
      IF(ISBLANK(SELECTEDVALUE(DimCity[city])), "Amenity Type and Amenities Average price in " & (SELECTEDVALUE('Calendar'[Year])), 
      IF(ISBLANK(SELECTEDVALUE('Calendar'[Year])), "Amenity Type and Amenities Average price in " & (SELECTEDVALUE(DimCity[city])),
       "Amenity Type and Amenities Average price in " & (SELECTEDVALUE('Calendar'[Year]) & " in " & (SELECTEDVALUE(DimCity[city]))))))</pre>

</details>

<details>
  <summary><strong>Power BI Report, Page 4</strong></summary>

  🌟 Number of reviews:
  <pre>Review_number = COUNT(Reviews[review_id])</pre>

  🌟 Number of reviewers:
  <pre>Reviewers_number = DISTINCTCOUNT(Reviews[reviewer_id])</pre>

  🌟 Total listings with reviews:
  <pre>review_listing_number = DISTINCTCOUNT(Reviews[listing_id])</pre>

  🌟 A measure for dynamically color-coding the top season for each city in the visual 
  **City's Top Season by Number of Reviews**:
  <pre>Color_SeasonForCity = 
  VAR MaxReviewPerSeason = 
  CALCULATE(
    MAXX(
        ALL(Calendar_review[Season]),
        [Review_number]
    ),
    VALUES(DimCity[city]))
  
  VAR CurrentValue = [Review_number]

  RETURN
  IF(CurrentValue = MaxReviewPerSeason, 1, 0)</pre>

  🌟 A measure for dynamically color-coding the Number of reviews in **Number of reviews Heatmap**:
  <pre>Normalized_Review = 
  VAR CurrentYear = SELECTEDVALUE(Calendar_review[Year])
  VAR MonthlyReviewCounts =
    SUMMARIZE(
        FILTER(
            ALL(Calendar_review),
            Calendar_review[Year] = CurrentYear
        ),
        Calendar_review[Month],
        "ReviewCount", CALCULATE(COUNT(Reviews[review_id]))
    )
  VAR MinValue = MINX(MonthlyReviewCounts, [ReviewCount])
  VAR MaxValue = MAXX(MonthlyReviewCounts, [ReviewCount])
  VAR CurrentValue = [Review_number]
  VAR Scale =
    IF(
        MaxValue = MinValue,
        0,
        DIVIDE(CurrentValue - MinValue, MaxValue - MinValue)
    )
  RETURN
  SWITCH(
    TRUE(),
    ISBLANK(Scale), BLANK(),
    Scale <= 0.05, "#FFD2D6",   
    Scale <= 0.15, "#FCC6C9",   
    Scale <= 0.30, "#ffb3b6", 
    Scale <= 0.45, "#ff7a80",   
    Scale <= 0.60, "#ff5a5f",
    Scale <= 0.75, "#d94d54",   
    Scale <= 0.90, "#b83e44",   
    "#A1343C"                   
  )</pre>
</details>

<details>
  <summary><strong>Power BI Report, Page 5</strong></summary>

  🌟 Calculated column `Price_per_guest` in `Listings`:
  <pre>Price_per_guest = DIVIDE(Listings[Price_in_usd], Listings[accommodates])</pre>
  
  🌟 Average price per guest:
  <pre>Price_per_guest_AVG = AVERAGE(Listings[Price_per_guest])</pre>
  
  🌟 Average review scores value:
  <pre>Review_scores_value_AVG = AVERAGE(Listings[review_scores_value])</pre>
  
  🌟 Min price per guest:
  <pre>MIN_price_per_guest = MIN(Listings[Price_per_guest])</pre>
  
  🌟 Max price per guest:
  <pre>MAX_price_per_guest = MAX(Listings[Price_per_guest])</pre>
  
  🌟 City rating: create a composite "value" metric, that gives a normalized score of rating per unit of cost
  <pre>Norm_value_score = DIVIDE([Review_scores_value_AVG],[Price_per_guest_AVG])</pre>
  
  🌟 Best city for travel:
  <pre>Best_city_for_travel = 
  VAR TopCityTable = TOPN(1, ALL(DimCity), [Norm_value_score], DESC)  
  RETURN 
  CONCATENATEX(TopCityTable, DimCity[city], ", ")</pre>
  
  🌟 Dynamic title for Best city for travel by years:
  <pre>Best_city_for_travel_by_years = 
  IF(SELECTEDVALUE('Calendar'[Year])=BLANK(),
     "Best City for Travel: ", 
     "Best City for Travel in " & (SELECTEDVALUE('Calendar'[Year]) & ": ")
  )</pre>
  
  🌟 Dynamic title for Average price per guest by years:
  <pre>Average_price_per_guest_by_years = 
  IF(SELECTEDVALUE('Calendar'[Year])=BLANK(), 
     "Average Price per Guest", 
     "Average Price per Guest in " & (SELECTEDVALUE('Calendar'[Year]))
  )</pre>
  
  🌟 Dynamic formatting for scatter plot
  
  - City with MAX Review_scores_value_AVG:
  <pre>TOP MAX City by Review_scores_value_AVG = 
  CALCULATE(MAXX(
      TOPN(
          1,
          ADDCOLUMNS(
              SUMMARIZE(ALLSELECTED(Listings), Listings[city]), 
              "AvgValue", [Review_scores_value_AVG]
          ),
          [AvgValue], DESC
      ),
      Listings[city]
  ), ALLSELECTED(Listings))</pre>
  
  - City with MIN Price_per_guest_AVG:
  <pre>TOP MIN City by Price_per_guest_AVG = 
  CALCULATE(MAXX(
      TOPN(
          1,
          ADDCOLUMNS(
              SUMMARIZE(ALLSELECTED(Listings), Listings[city]), 
              "AvgValue", [Price_per_guest_AVG]
          ),
          [AvgValue], ASC
      ),
      Listings[city]
  ), ALLSELECTED(Listings))</pre>
  
  - Measure to assign colors to data points in the 'Markers' section of the Scatter Plot formatting pane.
  <pre>TOP_2_categories = 
  IF(
      SELECTEDVALUE(Listings[city]) = [TOP MIN City by Price_per_guest_AVG] || 
      SELECTEDVALUE(Listings[city]) = [TOP MAX City by Review_scores_value_AVG], 1, 0)</pre>

</details>

## Design and development of visuals
The report includes:

- Slicers by `Year` and `City`.
- Visuals such as Cards, Bar Charts, Line and Column Charts, Scatter Plots, Stacked Charts, Treemap, Funnel Charts, and Matrices.
- Multiple bookmarks to place more visuals on a single page.
- Custom tooltips for some visuals to provide extended information beyond what is shown in the main visual.
- Custom bookmarks for the `City` slicer related to the `Map` visual. When no city is selected, 
the Map displays the total number of listings by city (the bubble size represents the number of listings 
in each city). If a specific city is selected, the Map visual — using a bookmark — shows the individual 
listings in the selected city.
- Custom parameter to select and display the TOP amenities based on the number chosen in the parameter.
- Dynamic visual titles depending on the selected Year and City, as well as measures for dynamic 
color formatting of visuals, including the Scatterplot and HeatMap.

## Question 1: Can you spot any major differences in the Airbnb market between cities?

According to visuals builded in the pages 1-3, it seems the major differences in the Airbnb market between cities as follows:

### Listings Quantity:

- Paris has significantly more listings, suggesting a larger or more competitive Airbnb market.
- Bangkok has the lowest number of listings over the entire observation period.
- The largest increase in new listings occurred in 2015, while since 2020 there has been a decline in the growth of new listings, probably due to COVID-19.

### Average price per Listing:
- Rio de Janeiro has the highest average price per Listing over the entire observation period.
- Four cities — Rio de Janeiro, Cape Town, Sydney, and Istanbul — have an average price per listing that is higher 
than the overall average price across all cities and all years.
-  Year 2009 has the highest average price over all years, and the most expencive city with the highest average price in 2009 was Istanbul.

### Most visited city:
- The most visited city by number of accomodaties over the entire observation period is Paris.

### City Rating:
- The city with the highest ratings over the entire observation period is Mexico City.
- The city with the lowest ratings over the entire observation period is Hong Kong.

### Room Types:
- "Entire place" Room Type has the highest number of listings across all cities and all years,
while the lowest number of listings is in the "Shared room" category.
- The distribution of room types varies in cities, some cities favor entire apartments, others have more private rooms.
- In Paris, the "Entire place" category holds the largest share.
- In New York, Mexico City, and Istanbul, the "Entire place" and "Private room" categories have nearly equal shares. 
- In Hong Kong, "Private room" is the dominant category.

### Amenity availability :
- Amenity availability also differs, possibly reflecting local housing norms or guest expectations.
- The most common amenity among all listings across all cities and years is Wi-Fi. 
- However, when looking at individual cities, the most common amenity in Bangkok and Hong Kong is Air Conditioning, 
in Paris and Rome – Heating, in Cape Town - Essensials, and in Rio de Janeiro and Sydney – Kitchen. 
In the other cities - most common amenity is Wi-Fi.
- The highest average price for all listings per all cities and years is for listings with the amenity "Air Conditioning".
- The highest average price in Bangkok and Cape Town is for listings with the amenity "Pool",
in Hong Kong - "Refrigerator", in Istanbul - "Smoke Alarm", in Mexico City and New York - "Dryer", 
in Paris - "Coffee Maker", in Rio de Janeiro and Sydney - "Free parking on premises", 
and in Rome - "Fire extinguisher".


## Question 2: Which attributes have the biggest influence in price?
As a result of the analysis, no significant influence of any specific category or attribute
on the price was identified: in different years and different cities, the price is affected by various independent factors.
So, there is no single attribute that consistently influences price across all locations and years.
Influential factors vary depending on the city, year, and market conditions.

## Question 3: Are you able to identify any trends or seasonality in the review data?
Yes, as shown on report page 4, the Airbnb review data exhibits seasonal patterns, 
both globally and at the city level.

These trends likely reflect local tourism patterns and seasonal travel preferences.

Across all cities and years, **the Autumn season consistently receives the highest number of reviews**, 
making it the top-performing season globally in terms of user reviews activity.

City-Level Seasonality Differences:
- In **Bangkok**, **Cape Town**, **Hong Kong**, **Mexico City**, **Rio de Janeiro**, **Sydney** the TOP Season by Review number is **Winter**
- In **Paris** the TOP Season by Review number is **Summer**
- In **Istanbul**, **New York** and **Rome** the TOP Season by Review number is **Automn**

This can be explained by the following reasons:
- Geographic hemisphere: the cities are located in a different geographic hemisphere (e.g., Sydney, Cape Town and Rio de Janeiro 
are in the Southern Hemisphere)
- Winter peaks in cities like Bangkok, Cape Town, and Sydney suggest these are warm-weather destinations favored by travelers during their home country’s colder months.
- Cultural travel patterns (e.g., European cities peaking in Summer or Autumn).
- Summer popularity in Paris fits the traditional European vacation season.
- Autumn popularity in New York, Istanbul, and Rome may reflect shoulder-season travel — when crowds and prices drop after summer.

So, while Autumn is the TOP Season globally for reviews, city-level trends vary.
- Bangkok, Cape Town, Hong Kong, Mexico City, Rio de Janeiro, and Sydney peak in **Winter**,
likely due to their climate appeal during colder months elsewhere.
- Paris sees a spike in **Summer**, in line with European travel trends, 
- Istanbul, New York, and Rome peak in **Autumn**, possibly due to milder weather and fewer crowds. 


## Question 4: Which city offers a better value for travel?
Better value for travel usually refers to getting more for less:
- Lower prices relative to the quality or rating;
- High satisfaction (ratings) for lower prices.

As shown on report page 5, Bangkok offers the best overall value for travel across all observed years, 
based on the normalized value score (measure **Norm_value_score**), which considers both 
average review value scores and average price per guest.

* Best City by Normalized Value Score is Bangkok.
* Best City by Lowest Average Price per Guest is also Bangkok.
* Best City by Highest Average Review Score (Value) is Mexico City.

So, Bangkok stands out as a destination that provides great experiences at a relatively low cost, 
which may reflect favorable local pricing, strong tourism infrastructure, or both.

Mexico City indicating high guest satisfaction, but may have slightly higher prices, 
which reduces its normalized score.

Bangkok emerges as the top city in terms of overall value, offering both the lowest average price per guest and 
a strong average review score. 
While Mexico City received the highest average review score for value, 
its slightly higher cost per guest makes it rank lower in normalized value. 

These insights suggest that Bangkok provides the most cost-efficient travel experience among 
the analyzed cities.
