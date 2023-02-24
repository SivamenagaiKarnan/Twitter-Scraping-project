# Twitter-Scraping-project
#Scraping of Twitter data using snscrape and storing it as a dataframe
#Installing the modules streamlit and pymongo(MongoDb)

pip install streamlit
pip install pymongo

#Importing the necessary modules

import streamlit as st
import snscrape
import snscrape.modules.twitter as sntwitter
import pandas as pd
import datetime
import pymongo
from pymongo import MongoClient

#Creating a title and subheader for displaying

st.title('Twitter scraping') 
st.subheader('Here we are going to scrape data from twitter using snscrape and storing  it in a dataframe')

#Creating the variables to store username or hashtag, number of tweet counts, date time etc., 

username=st.text_input('Enter the keyword or Username')
number=st.slider('Count the number of tweets',0,5000)
start=st.date_input('Enter starting date')
enddate=st.date_input('Enter ending date')
click=st.button('click here to search')

# Creating list to append tweet data to
tweets_list = []

# Using TwitterSearchScraper to scrape data and append tweets to list, here we are using for loop and append function to append the data into the list.
for i,tweet in enumerate(sntwitter.TwitterSearchScraper(f'{username} since:{start} until:{enddate}').get_items()):
    if i>number:
        break
    tweets_list.append([tweet.date, tweet.id, tweet.content, tweet.user.username])
    
# Creating a dataframe from the tweets list above

tweets_df = pd.DataFrame(tweets_list, columns=['Datetime', 'Tweet Id', 'Text', 'Username'])
data=st.dataframe(tweets_df)
print(data)

# Now we are defining a function to convert the dataframe created above to a csv file.
@st.cache
def convert_df_csv(df):
    # IMPORTANT: Cache the conversion to prevent computation on every rerun
    return df.to_csv().encode('utf-8')
csv = convert_df_csv(tweets_df)

# Creating a button to download csv file.

cs=st.download_button(
    label="Download data as CSV",
    data=csv,
    file_name='tweets.csv',
    mime='text/csv',
)

# Now we are defining a function to convert the dataframe created above to a json format.
@st.cache
def convert_df_json(df):
    # IMPORTANT: Cache the conversion to prevent computation on every rerun
    return df.to_json().encode('utf-8')
json = convert_df_json(tweets_df)

# Creating a button to download json file.

cs=st.download_button(
    label="Download data as json",
    data=json,
    file_name='tweets.json',
    mime='text/json',
)
# Making a Connection with MongoClient
if st.button('Upload'):
    client = MongoClient("mongodb://localhost:27017/")
# creating a database
    db = client["tweets"]
# creating a collection inside the database
    collection= db["tweet_datas"]
    tweets_df.reset_index(inplace=True)
# Converting dataframe into a dictionary, so that the dictionary can be easily converted into a Mongodb database   
    data_dict=tweets_df.to_dict('records')
    collection.insert_many(data_dict)
 else:
    st.write('None')

#The file gets directly stored into the mongodb database.

