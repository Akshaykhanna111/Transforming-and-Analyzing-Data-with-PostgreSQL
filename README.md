# Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
The objective of this project is to study the data of an ecommerce platform and derive insights from it that can help make strategic business decisions and highlight issues pertaining to customer experience. The data points provided for analysis comprises of web session details, user activity logs and product related information like inventory turnover ratio and customer sentiment. It's an open ended problem statement with few specific scenarios wherein particular information is required from the data analyst regarding revenue across categories, cities and countries. 

## Process
Below I have entailed the methodology adopted while working on this project - 
1. Data understanding - Study the 5 files shared for this project and go through each attribute to get a flavor of what data points are available for analysis.
2. Go through the questions shared for the project to identify the key parameters on which insights are required. Also, identify additional data points that might be of high significance from an analysis standpoint.
3. In the current form and shape validate whether any data normalization is required or the current table structures are already 3NF compliant. 

## Results
The web traffic data provided for this project can be used to derive critical insights on various aspects that can help a company improvise their customer experience and reduce their operational costs. Some of the aspects covered in my analysis revolved around - 
1. Avg and Total Revenue across regions and categories
2. Product seasonality trend analysis
3. Channelgroup wise number of visits and their respective conversion rates
4. Top and Bottom 10 products as per the customer sentiment score
5. Web traffic trend analysis by day, month and year.
6. Identifying on which pages most of the drop is happening in the customer funnel and optimizing the same
7. Evaluating the efficacy of search keywords as what's been seen is majority of traffic is organic but the conversions through this channel is very low compared to direct and referral marketing campaigns.  

## Challenges 
There were multiple issues with data. Normalization and referential integrity norms could not be followed at all with the given form and shape of data. Lots of columns that could be of use for critical analysis, had more than 50% records missing. Data had lot of redundancy and duplication, for example same non prime attributes were common across files, single SKUs had multiple different descriptions and categories in the sessions data, product SKUs in sessions table were not matching with those in products table, only 81 records were shared from sessions data where a transaction flag was 1, rest all were 0 out of the volume of 15K records and lastly columns like category, city, country etc had lots of junk values.  

## Future Goals
Following are the the points that should be accorded priority going forward for effective analysis - 
1. Normalization of database with referential integrity and accuracy constraints in place.
2. More refined schema design in place with additional data points available for analysis like supplier and shipment related data, or customer demographic data etc. 
3. Fixing issues with data collection, ideally junk values should not be getting captured and if so then there has to be standardization to club all into a single category rather than having different kinds of junk values across tables for the same column.  
4. Having an effective QA framework in place, which should be scheduled for execution at regular fixed periods to keep monitoring data related issues.
5. Employing user defined functions and views to automate reporting framework for various stakeholders
6. Setting up a robust data pipeline to ensure seamless data collection on a daily basis with guardrails implemented in the form of a sturdy QA framework
7. Doing more in-depth analysis on customer funnel, marketing campaigns ROI, changing customer personas and preferences across geographies and time. 
