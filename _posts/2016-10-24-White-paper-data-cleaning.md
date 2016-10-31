---
layout: post
title: White paper about the philosophy of 'Clean Data'
author: Gregory Vial
Categories: GeneralAssemblyProject
Tag: whitepaper, datacleaning
---

### Motivation
It said that a proportion as large as 80% of data analysis is spent on cleaning the data to make it suitable for the analysis itself. This mandatory but low value-added activity diverts the analysts attention from activities that matter most: building efficient predictive models and providing insights to data that will help executives make sensitive business decisions.

This paper describes the two essential steps of data cleaning called data scrubbing (work on content) and data tidying (work on structure). The purpose is to help perform these activities with maximum awareness of what they entail, but also reduce time spent on them.

### Data scrubbing
Data scrubbing focuses on the content of the data.
Its aim is to detect and correct (or remove) missing, corrupted or inaccurate data.

To be considered "clean", the data must comply to the following rules:
* Validity: data complies to a set of constrains, such as data type (is an integer, a string, a boolean etc), range (appropriate categorical variable such as Female or Male), mandatory (no essential value missing), unique (in case of a table key, such as an order number or a client ID) etc
* Accuracy: degree of comformity of a measure to a standard - this is usually difficult to achieve through data cleaning as it may require to access and validate data sources
* Completeness: degree to which all required measures are known. When essential values are missing, to some extent one can use data imputing to fill the gaps, however this must be done taking into considerations the impact on the final results.
* Consistency: degree to which a set of measures are equivalent across systems. Do we use the same ID to track a customer across various systems? Am I using the same categorical variables to capture the customer sex: 'F' and 'M' in one system and 'Female' and 'Male' in another could lead to inconsistencies.
* Uniformity: degree to which a set data measures are specified using the same units of measure. Typically the use of miles vs kilometers or degrees farenheit vs celcius.

### Data tidying
Data tidying is a standard way of organizing your data so it is easy to process, analyse and link to other data. So the focus is the structure, not the content.

The expected characteristics of a dataset after data tidying are the following:
* Each variable forms a column
* Each observation forms a row
* Each table stores information about one and only one observational unit

Typically, data before tyding will present one or several of the following characteritics that need to be fixed:
* column headers are values, not variable names
* multiple variables are stored in one column
* variables are stored in both rows and columns
* multiple types of observational unit are stored in the same table
* a single observational unit is stored in multiple tables

### Process
The typical data cleaning process will include the following steps:
* Data auditing: detect anomalies. This can be done through statistical methods, manual checks, automated verifications...
* Workflow specification: design a sequence of activities required to remove or correct
* Workflow execution: once workflow is designed and validated, develop appropriate code and execute it
* Post-processing and controlling: check the data is in the expected format. If not, loop back to the first step.

### Other considerations
The following are best practices that you should consider implementing
* It is essential to consider reproducibility of any of the cleaning steps performed. It enables others understand the process, reproduce your work and validate it. It may be useful to track down errors in results linked to inappropriate removal of outliers or impute of missing values, or instance.
* In general it must be kept in mind that the first step to high data quality is corporate culture and processes. Clean data is not only a data analyst task, it is down to the whole organisation to enable it. The better input quality, the less cleansing required, and the better output.

### References
<a href="https://en.wikipedia.org/wiki/Data_cleansing">Data cleansing (Wikipedia)</a>

<a href="http://vita.had.co.nz/papers/tidy-data.pdf">Tidy data (Journal of Statistical Software)</a>
