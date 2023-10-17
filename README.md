# Goal of the Project

In this project, I explore the concept of bias in data using Wikipedia articles. I try to acquire data relating to Wikipedia article searches for some cities/towns across the US and what the quality of the Wikipedia articles for these cities is. I also try to analyze some important metrics corresponding to the predicted quality, such as the states with best per capita coverage of articles. The aim is to understand the statewise trends in article coverage/presence of high quality articles.

# Source Data

1. A csv file **us_cities_by_state_SEPT.2023.csv**, containing the state, page title (city_state) and url. Data is to be acquired for these titles. Has been crawled from https://en.wikipedia.org/wiki/Category:Lists_of_cities_in_the_United_States_by_state.

https://drive.google.com/file/d/1XAydF2Cqjr5u1zs-B9p09JVliqtFYv15/view?usp=drive_link

2. An excel file **NST-EST2022.xlsx**, reworked from the original census data (https://www.census.gov/data/tables/time-series/demo/popest/2020s-state-total.html). This file contains the list of states and the 2022 population estimates for each state.

3. A csv file **US States by Region - US Census Bureau - Sheet1.csv**, containing regions (Northeast, Midwest etc.), divisions (New England, Middle Atlantic etc.) and states (Connecticut, Vermont etc.)

https://drive.google.com/file/d/1uG6Pj5m3NjBbx9Xkzdtfo_Ewo0zCNK8F/view?usp=drive_link

# Licensing

1. Wikimedia Foundation REST API terms of use : https://www.mediawiki.org/wiki/API:REST_API#Terms_and_conditions

2. Creative Commons License: https://creativecommons.org/licenses/by/4.0/

# Important Considerations for the Data

The dataset is massive and has around 22000 articles that need to be analyzed. It takes a long time to access the API, hence the data acquisition process needs good error handling. The data has duplicate entries, which can be deleted by storing entries into a dictionary, which does not take duplicate keys. Later on during the analysis process, I realized that there are some articles which are not geographic locations,(US Census etc.) which I had to handle and remove separately. Population data has entries like (District of Columbia0 which were also removed in the data merging process.

# Helpful API Documentation

1. ORES Wiki : https://www.mediawiki.org/wiki/ORES
   
2. Wikipedia Content Assessment: https://en.wikipedia.org/wiki/Wikipedia:Content_assessment
   
3. Wikimedia API Documentation: https://www.mediawiki.org/wiki/API:Info
   
4. Python Notebook to call Wikipedia API to get article metadata: https://drive.google.com/file/d/15UoE16s-IccCTOXREjU3xDIz07tlpyrl/view?usp=sharing
    
5. Python Notebook to call ORES ML model to get article quality predictions: https://drive.google.com/file/d/17C9xsmR9U3lJeD52UTbAedlHDetwYsxs/view?usp=sharing

# Coding Process

1. In the Python Notebook **DATA 512 A2 Data Acquisition.ipynb**, we first read in data from **us_cities_by_state_SEPT.2023.csv**. We set up API parameters and the function to call the Wikipedia API. Calling the API takes some time but we generate the data containing article titles and last revision id, which is stored into an intermediate file **city.json**.

2. Reading in **city.json**, we get the list of article titles and the corresponding revision id. The API call to the ORES model is also defined here and called iteratively. An access token is generated from the same from the Wikipedia account, which has been hidden here for security reasons. The article title and predicted quality is saved in a dictionary. This takes a long time to run (5-6 hours). The API also times out from time to time and needs to be rerun from the point it has stopped at. Finally, we acquire the list of predicted qualities, which is stored into **predictions.csv**.

3. Now we need to merge different data fields together. First, we split article in **predictions.csv** to separate into city,state and save the states. Handle some cases wheere there is no state separately. Read in the **US States by Region - US Census Bureau - Sheet1.csv**, conjoin region and division fields into 'regional_division' and map it to each state. Then we can merge this with the dataframe from **predictions.csv** and get a combined dataframe with region data. After that, we read in population data from **NST-EST2022.xlsx** and merge it with previous dataframe to get a combiend dataframe with population data. Finally, we read in the intermediate **city.json**, get the last revision ids, and merge that with previous dataframe to get a final dataframe with all required fields. Remove excess fields, rename certain columns, and reorder them. Save this to **wp_scored_city_articles_by_state.csv**. This final csv has following data fields as requested:

| Fields    |
| -------- |
| state  |
| regional_division |
| population    |
| article_title |
| revision_id |
| article_quality |

4. In the Python Notebook **DATA 512 A2 Data Analysis.ipynb**, we read in the **wp_scored_city_articles_by_state.csv** and carry out the required analyses. For the 1st and 2nd questions, we group the data by state, getting the count of article titles and mean of populations. This can be divided to get per capita total articles per state. Sorting this in descending order will answer Q1 and sorting it in ascending order will answer Q2. Q3 and Q4 are answered similarly, but the dataframe needs to be initially filtered to include only *high quality* articles, i.e. with predicted quality *Featured* (FA) or *Good* (GA). Same analysis can then be used as Q1,2. For Q5,6 the process differs slightly. We must first groupby state, to get total number of articles per state, population of each state and regional division of each state. Then we can group by regional division and calculate the sum of population and article count to cover all articles in the given division. This can be divided to get per capita information. Sorting in descending order will answer Q5. Carrying out the same rpocess but only with high quality articles will answer Q6.

# Research Implications

This assignment gave me some interesting perspectives into Wikipedia, as a data and information source. First off, the concept of quality rankings of Wikipedia articles was new to me and put forth intriguing observations about which articles are better quality and why. Some intersting obervations were that the coverage of articles per capita was often better for more obscure states such as Wyoming, vermont, The Dakotas etc. This can be attributed to the relatively lower population in these states and intuitively makes sense. Similarly, there are more higher ranked articles per capita for these states rather than larger states such as Florida, California, New York etc. A counter intuitive finding for me was that major cities, such as San Francisco, Seattle, New York City do not have very highly rated articles. This surprised me becasue these articles are extremely detailed, but still not high quality. It is probably because these articles are edited so frequently that it cannot be trustworthy at any given point of time.

Some biases I observed in the data were as mentioned above, population centers make a big difference to rankings per capita. Major cities also display contrasting qualities of articles, probably because of the frequent edits. Major cities are also named differently, not having state names included in the article, hence they need to be handled differently. During the process of acquiring and analyzing the data, I was pleasantly surprised to see that there was absolutely no missing data after the ORES calls(which was something I expected). This proves that the ORES model is really scalable and will provide results consistently. I would love to study more about the workings of the ORES model to understand better, how it actually grades articles, the weightage of different parameters and the mathematical considerations behind assigning it a probability score.

Wikipedia as a data source has lots of potential quite simply because of the vast amounts of data it contains. This assignment was simply based on getting metadata relating to article, and still there is so much analysis we could carry out. Wikipedia is colloquially considered to be a relaible data source and is the foremost repository of information online. However, at the crux of it, it is open source and any members of the general public can make edits. This is often problematic and malicious agents can mess up our data sources, causing factual inaccuracies. This has in the past led to edit wars, and has ultimately ended in some articles being locked and uneditable (like San Francisco). Despite this, I personally believe Wikipedia is a great data source for various forms of analysis. The previous assignment helped us analyze page views, this one analyzed article qualities, and there are many such use cases which can be taken up by data analysts.

There are always potential issues in using open source data such as Wikipedia for data science research. Data on the internet has an incredible amount of biases, ranging from geographical, racial, color based, economy based and so much more. We saw this in our assignment where bigger population centers influenced article quality. Algorithms trained on such biased datasets will be skewed towards giving a particular result, and this may mess up our analysis. Overrepresented groups or regions can heavily impact data driven decisions and hence decision making baserd on data science research must be carried out very cautiously, else there could be detrimental effects in the real world.

However, there are a lot of use cases where Wikipedia could support data science research really well. Wikipedia is generally very reliable for historical data, hence time series analytics is a major use case. The ubiquituousness of Wikipedia makes it the primary resource of information online, quite simply because it has a plethora of information available. The Wikimedia foundation has now started providing great APIs to access more and more aspects of data and metadata, and models to unerstand this data better. All these resources make data science on Wikipedia data an attractive proposition, and I certainly can attest to this as a budding data scientist!
