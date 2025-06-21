#                                                                             <p align="center"> Tuning Up Music Highway</p>
This is a repository for the Tuning Up Music Highway project for The Erdős Institute Summer 2025 Data Science Bootcamp.

Team members: Ruixuan Ding, [John Hurtado](https://github.com/hurtadocadavid21), Yang Mo, [James O'Quinn](https://github.com/JamesOQ), [Chilambwe Wapamenshi](https://github.com/ChilambweWapamenshi)

Aknowledgements: First, we would like to thank everyone at The Erdős Institute for making this project possible and giving us the means, structure, and education to complete this project. Additionally, we would like to thank the Tennessee Department of Safety and Homeland Security for providing their detailed data and well-made dashboards to the public. Also, we would like to thank Steven Gubkin for the helpful comments and feedback. Finally, we are incredibly grateful for all the advice, encouragement, and guidance from our group mentor Greg Taylor.
##  <p align="center"> Introduction</p>
&nbsp;&nbsp;&nbsp;&nbsp; Considered by many residents, and even some experts ([here](https://www.dangerousroads.org/north-america/usa/10683-i-40-the-most-dangerous-road-in-nashville-for-auto-accidents.html) and [here](https://www.gkbm.com/blog/interstate-40-car-accidents-tennessee/)), as the most dangerous highway in Tennessee, Music Highway, the stretch of Interstate 40 between Memphis and Nashville, could use a serious tuning up. The goal of this project is to use data modeling and hypothesis testing to determine the efficacy and cost-effectiveness of various safety strategies to implement on actual segments of Music Highway in Madison and Henderson counties. We accomplish this by procurring  a recent crash dataset from the Tennessee Department of Safety and Homeland Security which includes both crash severity and GPS location of the crash, filtering out crashes from the dataset which did not occur on the section of I-40 that pass through Madison and Henderson counties, tagging the crashes in the dataset with geospatial features (presence of guardrails, type of median, pavement conditon, etc.) manually recorded from Google Maps, doing some exporatory data analysis to determine the best strategies and target segments, performing model-based hypothesis testing to determine the efficacy of various safety strategies on crash severity reduction, and, finally, creating a cost-benefit analysis for the strategies most supported by the data.

## <p align="center"> Dataset Generation</p>
&nbsp;&nbsp;&nbsp;&nbsp; Since physical safety strategies are the type of strategy most feasible for our modeling approach, we set out to create a dataset which would allow us to understand how severity of a crash (in terms of human harm) would be influenced by the various geographic and physical features of the highway near the crash. The first step we took is that we collected recent (2023-2025) crash data from the dashboard publicly provided by the Tennessee Department of Safety and Homeland Secuirty:
1. https://www.tn.gov/safety/stats/dashboards/recent-crashes.html
2. https://www.tn.gov/safety/stats/dashboards/fatalseriousinjurycrashes.html

This involved highlighting the crash data we wanted to collect on the dashboards with a selection box and saving the data as a csv file. Since the "Recent Crashes" webpage would crash if too many datapoints were selected at once, and noticing that the stretch of I-40 going through Madison and Henderson counties had an unsually high number of crashes with severe injuries and fatalities given that they are mostly rural, we decided to focus only on the crashes that occured on I-40 in Madison and Henderson counties. While it was infeasible to collect only the crashes from the "Recent Crashes" dashboard that happened on I-40, we were able to get all recent crashes in Madison and Henderson counties and resolved to filter for I-40 crashes later. Here, are the two datasets immediately after we collected them from the above dashboards:
1. [Recent Crashes in Madison and Henderson Counties](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/datasets/Overall%20geospatial%20crash%20data%20for%20Madison%20and%20Henderson%20counties.csv),
2. [Serious Injuries and Fatalities I-40 Dataset](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/datasets/Serious%20Injuries%20and%20Fatalities%20Data%20for%20I-40%20Tennessee.csv).

Note that the first dataset only includes three features: latitutde, longitude, and crash type. While we did reach out to the responsible Tennessee state department to see if we could get a "Recent Crashes" dataset as detailed as the "Serious Inuries and Fatalities" dataset and recieved a reply, we never received that more detailed dataset.

&nbsp;&nbsp;&nbsp;&nbsp; Next, since the exact time and date of most crashes in the "Serious Inujuries and Fatalities" dataset were recorded, but not the weather condition and road wetness at the time of each crash, we set out to create a data pipeline which would query a weather API to automatically add weather information to each crash datapoint. We did this by creating a Python notebook which queries the Visual Crossing weather API to update our dataset with the corresponding weather condition at each crash. That notebook can be found [here](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/Code/Weather%20data%20API%20query.ipynb). We also added a graded road wetness feature by finding the cumulative rainfall total six hours before the time each crash occurred using the grading:
- more than 1 inch of cumulative rainfall is labeled as 'very wet'
- between 0 and 1 inches of cumulative rainfall is 'damp'
- otherwise 'dry'.

The notebook for that can be found [here](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/Code/Road%20Wetness%20API%20query.ipynb). While the weather and road wetness features were helpful during exploratory data analysis, we did not use these features in our final analysis.

&nbsp;&nbsp;&nbsp;&nbsp; The most important aspect of our feature engineering was the tagging of geospatial features manually recorded from Google Maps. Since all images on Google Maps of our segment on I-40 are consistent with the timeframe of crashes in our datasets (2023-2025) and no major construction projects occurred on this segment during that timeframe, we expect the geospatial features on Google Maps to be mostly consistent with the geospatial features at the time of each crash. We determined it would be best to manually record features through Google Streetview to maintain accuracy. The following table shows the features and categories that we recorded:
| Feature                                | 0                        | 1                        | 2                        | 3                        | 4                               |
|----------------------------------------|--------------------------|--------------------------|--------------------------|--------------------------|----------------------------------|
| **Presence of guardrails**             | None                     | One side only            | Both sides               | Partial or damaged       | –                                |
| **Cable barriers**                     | None                     | Median only              | Shoulder only            | Both median and shoulder | –                                |
| **Rumble strips**                      | None                     | Centerline only          | Shoulder only            | Both centerline & shoulder | –                              |
| **Pavement condition**                 | Unknown                  | Poor (cracks, potholes)  | Fair                     | Good                     | Excellent (recently resurfaced)  |
| **Proximity to entrances/exits**       | Not near entrance/exit   | Near entrance/exit (≤400m) | –                     | –                        | –                                |
| **Urban vs. rural**                    | Rural                    | Suburban                 | Urban                    | –                        | –                                |
| **Natural features**                   | Open/plain               | Forested/wooded          | Near water body          | Hilly/rocky              | Mixed/complex terrain            |
| **Number of lanes**                    | One lane                 | Two lanes                | 3–4 lanes                | 5+ lanes                 | –                                |
| **Shoulder type/width**                | No shoulder              | Narrow, unpaved          | Narrow, paved            | Wide, unpaved            | Wide, paved                      |
| **Posted speed limit**                | ≤35 mph                  | 40–55 mph                | 60–65 mph                | 70+ mph                  | –                                |
| **Median/divider type**                | None                     | Painted median           | Grass median             | Raised concrete divider  | Guardrail or cable divider       |
| **Lane marking/signage visibility**    | Missing/faded            | Poor                     | Average                  | Clear/visible            | Fresh/high-reflectivity          |
| **Nighttime lighting/visibility**      | No lighting              | Poor                     | Moderate                 | Well-lit                 | –                                |


The results of our geospatial feature recording can be found in [this google sheet](https://docs.google.com/spreadsheets/d/1zOUgfIgm1ztMX9Kwp7bK6PvghsTIMg_krHt-7JgNFRs/edit?gid=553882531#gid=553882531). Before tagging our datasets, we filtered our Recent Crashes dataset to only include the crashes which happened on I-40 using GeoPandas and the Tennessee roadways shapefile in [this notebook](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/Code/I40_Crash_Filter_GEOSPATIAL_JOIN.ipynb). Finally, we automatically tagged our datasets with the geospatial features we recorded with [this notebook](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/Code/Geospatial%20Automated%20Tagger.ipynb).

## <p align="center"> Exploratory Data Analysis</p>
&nbsp;&nbsp;&nbsp;&nbsp; For our exploratory data analysis, our goal was to find evidence for effective strategies to pursue and suitable segments to implement them on. We accomplished this through heatmap visualizations and feature importance rankings from preliminary modeling. The following charts, as well as others, were created in [this notebook](https://github.com/JamesOQ/Tuning-Up-Music-Highway/blob/main/EDA/Crash%20dataset%20visualizations.ipynb).

<p align="center">
  <img src="EDA/Crash Heatmap.png" width="700" alt="Logo" />
</p>

<p align="center">
  <img src="EDA/Crash Severity map.png" width="700" alt="Logo" />
</p>

We also did some preliminary modeling to get an idea of feature importance. These images, the first predicting fatalities and the second predicting injuries and fatalities, were generated from the notebooks here and here.
<p align="center">
  <img src="EDA/fatality feature importance.png" width="1000" alt="Logo" />
</p>

<p align="center">
  <img src="EDA/Injury feature importance.png" width="1000" alt="Logo" />
</p>

&nbsp;&nbsp;&nbsp;&nbsp; Since we determined that we would like to use a crash being injurious or not as the main feature we want to predict, it is very clear from the previous image that we are seeing a very large rural and urban divide. We removed all the ubran crashes from our dataset (The only urban area on our section of I-40 is Jackson, TN) and did some additional preliminary modeling and got the following graph. 

<p align="center">
  <img src="EDA/Jackson removed.png" width="1000" alt="Logo" />
</p>

Since it appears that there is a stark contrast between urban and rural crashes and rural segments of our highway are more targetable for safety interventions due to a combination of higher crash injury rate and lack of many of our features, we decided to remove the urban crashes (those in Jackson, TN) from the dataset to focus solely on implemeneting our safety strategies on rural segments. We created some interactive maps through html which can be accessed by clicking the images below.

&nbsp;&nbsp;&nbsp;&nbsp; Finally, we came up with a list of hypotheses to test.

- Adding guardrails and redoing lane markings and signage to a segment
- Only redoing lane markings and signage to a segment
- Repaving a segment
- Reducing the speed limit on a segment near a highway entrance/exit
- Adding dividers to the median of a segment
- Adding lampposts to a small segment

## <p align="center"> Modeling & Hypothesis Testing</p>
&nbsp;&nbsp;&nbsp;&nbsp; We will now outline our general approch to modeling and hypothesis testing. First, we selected the safety intervention to test from our list. Then, to choose which segment to test it on, we divded our overall highway into several segments of equal length and chose the segment with the best combination of
- high injury rate
- high proportion of targetable crashes by our safety intervention
- sufficient number of crashes in that segment for meaningful analysis
After, our segment was chosen, we held out all of the crashes that happened on that segment from our dataset and created training and validation sets from the rest. Since the balance between injurious and non-injurious crashes is quite lopsided, we used random oversampling to balance out the training set. Next, we trained the following 3 models:
1. Intepretable logistic regression model using only the features which correspond to our safety intervention
2. Full predictive logistic regression model
3. CatBoost model

&nbsp;&nbsp;&nbsp;&nbsp; We chose these models since our focus is on prediction and our data is purely categorical. Note that we added a Bayesian logistic regression model to the adding guardrails and redoing lane markings and signage hypothesis test but did not include it in the final analysis or any other hypothesis tests due to the following reasons:
1. The model took over 10 hours to train on our data due to running many Markov Chain Monte Carlo simulations to simulate several unknown priors.
2. The model lacked predictive power even with some good empirical priors
3. The model was difficult to tune and didn't provide the uncertainty quantification we had hoped for.
We also performed automatic tuning through a grid search for our CatBoost model and did some threshold selection using F1 score for both models. Even after this, our models still predicted a high number of false positives for injurious crashes and had average F1 scores around .5. However, we mostly settled for good recall because we would rather overpredict injuries than under when considering safety.

&nbsp;&nbsp;&nbsp;&nbsp; Following the training and tuning of our models, we ran our hypothesis test for the chosen safety intervention as follows. Note that the null hypothesis for each safety strategy is that the strategy does not lead to a reduction in the number of injurious crashes. First, we ran the models on our held-out segment data, first as-is then followed by changing the feature categories on all datapoints to match the safety intervention. Then, we compared the before and after predictions of our model and tested the hypothesis through bootstrap testing. That is, we create 10,000 sample differences in before and after means from our models by boostrapping a before and after dataset, finding a difference in their means, then calculating a p-value for each model by finding the ratio of sample difference means less than or equal to 0. If 0 was outside the confidence interval for both models, we rejected the null hypothesis.

## <p align="center"> Results</p>
&nbsp;&nbsp;&nbsp;&nbsp; Before discussing our results, there is one thing we would like to make clear. Since we trained our models on nearly identical datasets (only a small number of segment specific crashes held out) across all of our hypothesis tests, we are more likely to run into type I errors due to the multiple comparisons problem. We would have addressed this by applying Bonferroni correction to our p-values. However, since our only statistically significant hypotehsis test was the first one performed, the correction was unnecessary. 

&nbsp;&nbsp;&nbsp;&nbsp; Overall, adding guardrails and redoing the lane markings and signage to the segment with longitude range -88.663 to -88.605 was the only strategy for which both of our models showed a statistically significant reduction in injurious crashes. In fact, the high effectiveness of adding guardrails in reducing injurious crashes is well documented in the roadway safety literature, so this result is not too surprising. Moreover, this implementation turned out to be cost effect. Indeed, if each injurious crash our madel would have predicted to have happened the previous 3 years had a 50% cahnce of happening, we get the following crash ebenefit analysis:



Do note that the actual number of crashes that happened on that highway segment is actually 25. 

## <p align="center"> Future Work </p>

## <p align="center"> Description of Repository</p>




Files Included:

datasets:
  1. Serious Injuries and Fatalities Data for I-40 Tennessee.csv
     - This is our main dataset which contains all fatal and serious injury causing crashes which happened on I-40 Tennessee between 2023 and 2025.
     
  2. Overall crash data for Madison and Henderson counties.csv
     - Secondary dataset which contains the geospatial coordinates and severity of crash for each crash that happened in either Madison or Henderson Counties between 2023 and 2025.
  
  3. Serious Injuries and Fatalities Data for I-40 Tennessee *with weather*.csv
     - An update to our main dataset which includes added weather condition data queried by data, time, and location from Visual Crossing.
  
  4. Serious Injuries and Fatalities Data for I-40 Tennessee *with weather and wetness*.csv
     - Another update to our main dataset which grades the wetness of the road at the time of crash based on the cumulative rainfall of the 6 previous hours. The grading is as follows:
       - more than 1 inch of cumulative rainfall is labeled as 'very wet'
       - between 0 and 1 inches of cumulative rainfall is 'damp'
       - otherwise 'dry'.

Code:
  1. Weather data API query.ipynb
     - Program that queries historical weather data from the Visual Crossing weather API based on location, date, and time.
  2. Road Wetness API query.ipynb
     - Program that queries cumulative precipitation amount from the Visual Crossing weather API based on the 6 hours before the crash. 
  3. Geospatial Automated Tagger.ipynb
     - Program which takes our excel sheet of manually tagged geospatial features from Google Maps and, for each feature and datapoint in our dataset, tags the category of that feature which corresponds to the longitude of the crash location of the datapoint.
