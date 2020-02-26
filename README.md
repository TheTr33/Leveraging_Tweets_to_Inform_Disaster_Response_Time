# Levarging Tweets to Inform Disaster Response Time  
Authors: Jillian Berman, Sree Gollakota, Zhan Yu 
## Contents 
- [Overview](#Overview)
- [Problem Statement](#Problem-Statement)
- [Executive Summary](#Executive-Summary)  
	- [Data Gathering and EDA](#Data-Gathering-and-EDA)
	- [Preprocessing and Modeling](#Preprocessing-and-Modeling)
	- [Extracting Locations with spaCy](#Extracting-Locations-with-spaCy)
	- [Coordinates Acquisition with herepy](#Coordinates-Acquisition-with-herepy)
	- [Mapping](#Mapping)
- [Conclusions and Limitations](#Conclusions-and-Limitations)
    - [Next Steps](#Next-Steps-Summary) 
- [Sources](#Sources)
## Overview 
Having a speedy emergency response time is a critical part of standard disaster management protocol. However, current GIS data can sometimes be slow to update, providing inaccurate travel routes and travel times. Our aim is to build a platform that leverages social media, specifically tweets, to see if we can locate unreported closures, verify location and then plot it, to better inform current GIS platforms. We will be using Hurricane Matthew, and the Florida city of Panama City Beach to gather data to train our model. 

## Problem Statement 
In order to better improve emergency response times during disasters, this project aims to leverage social media to identify real time road closures or damaged roads, and other blocked routes that may affect travel time, traffic lights, safety, etc. 

## Executive Summary

##### Data Gathering and EDA

Using the Python library GetOldTweets3, pulled tweets from October 1st, 2018, when the storm first began to develop, through October 15, 2018, a few days after it had passed. Tweets unfrequently have any location data stored. To get around that,  we used two different parameters to help narrow our tweets pulled. First, we searched for the area’s 511 accounts and pulled all the tweets from the time period. Next, we searched for keywords, like the name of the storm and potential hashtags that might identify the area of disaster, etc. to build our training dataset. In the end, since this is the data during which an event is occuring, it is this data that we mapped.  
The second method of collecting data is using the Tweepy, another python library. Using the tweepy library with the Twitter API we are able to collect tweets from the past 7 days. It would be this method of data gathering that you’d use in the event of an emergency.  We searched for tweets from the same usernames and keywords that we’d searched for to gather our historical tweets. When pulling tweets based off of our keywords search, given that it was years after the storm, there were less mentions of those keywords when there wasn’t an active disaster event ongoing. 

##### Preprocessing and Modeling

In order to prepare the data used in the classification model, we classified our tweets on the from the 511 accounts, where we know the tweets are about traffic conditions or closures, as our positive class and tweets pulled with no accounts specified, just the keywords, would be the negative class. Once both the positive class tweets and negative class tweets were combined to a common dataframe, the corpus of tweets was generally cleaned by removing all website links, removing non-letters, converting all letters to lower-case, and removing stop words from the tweets. Then, the tweets were lemmatized to contextually reduce morphological variation as opposed to stemming which would simply trim words to their approximate root words without accounting for context. <br>
Once the tweets in the corpus were cleaned, a pipeline and gridsearch was built. In order to quantify the frequency and importance of words used in a tweet, the tweets in the corpus were count-vectorized and TFIDF-vectorized in the pipelines before being fed into the models. The tweets scraped from the 511 accounts were fewer than the tweets pulled without specific account parameters which caused a class imbalance due to our traffic incident/emergency classification assumptions. Therefore, the models chosen for the gridsearch were the random forest classifier, and the AdaBoost classifier as both of those models have bootstrapping built in to account for class imbalances. Additionally, after vectorizing the corpus with either `CountVectorizer`, or `TfidfVectorizer` each unique occurence of a term is made to be a feature. Assuming independence of features in the model, this would cause our data set to have high dimensionality and therefore a Support Vector Machine for classification was also included in the gridsearch with a `gamma` = ‘scale’ (scaled gamma distribution).<br>
After checking the accuracy score for the best estimators on the `X_test` set and the `X_train` set, it was evident that the AdaBoost model, and the Support Vector Machine(SVM) yielded the highest scores on the test set when compared to the random forest which was to be expected. In order to choose the best model between the AdaBoost and the SVM for our use case, we compared the sensitivity and specificity of both models on the test set. While both models were able to classify a tweet as not an emergency extremely well, this was due to the dominant class in our data set being tweets not from the 511 accounts. Given this, both models had a specificity of .99.  To improve upon these models, you would want to optimize the sensitivity of the model, so that false positives would be minimized and traffic/emergency incident tweets would not be ignored as normal tweets. Therefore, the AdaBoost model with the count-vectorized corpus is the ideal model with a sensitivity of .86 compared to the SVM for classification with the count-vectorized corpus having a sensitivity of .69. <br>
Due to our classification assumptions of the dataset being that the positive class is any tweet scraped from the BayCountTMC, fl511_panhandl, and WJHG_TV accounts, the model is technically built to classify between tweets from those  know traffic accounts and tweets not from those accounts. Therefore, the model may not perform well on live tweets which will consist of tweets from all sorts of accounts that won't have the specific language used in the preformatted announcements and tweets from the 511 accounts.<br>

##### Extracting Locations with spaCy
spaCy is an open-source software library for advanced natural language processing, written in the Python and Cython. spaCy’s models can be installed as Python packages. The build-in spaCy model does not perform very well since in general there are only three location related entities: `GPE`, `LOC` and `FAC`. Whereas most of our traffic tweets reference other location types like mile markers.  Therefore, we trained our own spaCy model by providing many examples to meaningfully improve the system. The trained model provides us more detailed location labels which return columns : 'road', 'streets', 'interstate', 'mile_marker', 'bridge', 'lanes' and  'town'. After training we are able to extract addresses from some of the tweets.<br>


##### Coordinates Acquisition with herepy
The HERE Geocoder API is a REST (Representational state transfer) API that allows us to obtain addresses from locations' coordinates or coordinates from addresses or landmarks. We are using herepy which is a library that allows us to run HERE Geocoder API in Python. We also ran into limitations here, with the herepy not being able to return coordinates for features we extracted like mile markers. <br>

##### Mapping
Finally, in the mapping section we took the coordinates generated via the HERE Geocoder API, ran them through Shapely and into a GeoDataFrame for mapping with Folium. The final output is a map of Florida with the tweet locations we generated through SpaCy processing. <br>


## Conclusions and Limitations  
As mentioned above, the classification model used for distinguished traffic/emergency incident tweets was based on a naive assumption that tweets from the 511 accounts would be the positive class and tweets from non-specific accounts would be the negative class. While the assumption allows us to distinguish the tweets accurately in the testing dataset, the model will more realistically distinguish tweets by the 511 accounts from tweets not tweeted by the 511 accounts when applied to a corpus of live tweets. In order for the model to be able to classify between an emergency/ traffic tweet from a normal tweet more contextually, a more robustly classified training model from more accounts other than the 511 accounts would help. <br>
While there is valuable information that is being tweeted out during disasters, the fact that most tweets do not have location data attached to them, and also the variance of the way that people compose these tweets make it a somewhat difficult resource to leverage. It’s important to also recognize that cellular service or wifi connection could potentially not be available depending on the area and type of disaster that is occuring.

##### Next Steps Summary
- Improving classification assumptions to be more specific to the tweet context as opposed to generalizing 511 account tweets as positive class and others as negative class  
- Applying for a Google Maps API to integrate real time traffic updates, and point to point directions. 


## Sources 

- [Evacuation Routes Repository](https://github.com/DCapella/evacuation-routes) 
- [GetOldTweets3 Documentation](https://pypi.org/project/GetOldTweets3/) 
- [Tweepy Documentation](http://www.tweepy.org/)
- [HERE.com](https://www.here.com/)   
- [HerePy GitHub](https://github.com/abdullahselek/HerePy)   
- [Named Entity Recognition for spaCy](https://spacy.io/api/annotation#named-entities)  
- [Road Closure Evacuation Routes](https://github.com/fmanon/Road_Closures_Evacuation_Routes)  
- [spaCy NER Annotator](https://github.com/ManivannanMurugavel/spacy-ner-annotator)  
- [Training Basics for spaCy](https://spacy.io/usage/training)  
