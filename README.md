#Installing and importing all necessary modules

pip install snscrape

pip install pymongo

import streamlit as st

import snscrape


import snscrape.modules.twitter as sntwitter

import pandas as pd

import datetime

import pymongo

from pymongo import MongoClient

# Creating title and subheader for the project

st.title('Twitter scraping') 

st.subheader('Here we are going to scrape data from twitter using snscrape and storing  it in a dataframe')

# Creating list to append tweet data to

username=st.text_input('Enter the keyword or Username')

number=st.slider('Count the number of tweets',0,5000)

startdate=st.date_input('Enter starting date')

enddate=st.date_input('Enter ending date')

click=st.button('click here to search')

# Creating list to append tweet data to

tweets_list = []

# Using TwitterSearchScraper to scrape data and append tweets to list
for i,tweet in enumerate(sntwitter.TwitterSearchScraper(f'{username} since:{start} until:{enddate}').get_items()):
    
    if i>number:
        
        break
    
    tweets_list.append([tweet.date, tweet.id, tweet.content, tweet.user.username])
    
# Creating a dataframe from the tweets list above

tweets_df = pd.DataFrame(tweets_list, columns=['Datetime', 'Tweet Id', 'Text', 'Username'])

data=st.dataframe(tweets_df)

print(data)

# Defining a function to convert the dataframe into csv and json format and creating a button to download those formats

@st.cache

def convert_df_csv(df):
    
    # IMPORTANT: Cache the conversion to prevent computation on every rerun
    
    return df.to_csv().encode('utf-8')


csv = convert_df_csv(tweets_df)

cs=st.download_button(
    
    label="Download data as CSV",
    
    data=csv,
    
    file_name='tweets.csv',
    
    mime='text/csv',
)

@st.cache

def convert_df_json(df):
    
    # IMPORTANT: Cache the conversion to prevent computation on every rerun
    
    return df.to_json().encode('utf-8')


json = convert_df_json(tweets_df)

cs=st.download_button(
    
    label="Download data as json",
    
    data=json,
    
    file_name='tweets.json',
    
    mime='text/json',
)

# Making a Connection with MongoClient and creating a database and a collection.

if st.button('Upload'):
    
    client = MongoClient("mongodb://localhost:27017/")

    db = client["tweets"]

    collection= db["tweet_datas"]
    
    tweets_df.reset_index(inplace=True)
    
# Converting dataframe into a dictionary so that it can be transfered directly into mongodb  
    
    data_dict=tweets_df.to_dict('records')
    
    collection.insert_many(data_dict)
 
 
else:
    
    st.write('None')


