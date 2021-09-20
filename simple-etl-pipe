#!/usr/bin/python39
"""This pipeline takes MySQL data and puts it into mongoDB"""

# Author : Buchkevich 12/03/2018

# System libs
import copy

# Internal libs
import config

# External Libs
from pymongo import MongoClient
import pymysql


# Pipeline constants
RESET_MONGO_COLLECTIONS_ON_UPDATE = True # Resets the collections if a collection already exists, if false, the data is appeneded to the collection
PRINT_INFO = True # Print options for debugging purposes
PRINT_RESULTS = True # Print option for debugging purposes

def initalise_mysql():
    """Initalises and returns a MySQL database based on config"""
    return pymysql.connect(
        host=config.MYSQL_HOST,
        user=config.MYSQL_USERNAME,
        password=config.MYSQL_PASSWORD,
        db=config.MYSQL_DB)

def initalise_mongo():
    """Initalises and returns MongoDB database based on config"""
    return MongoClient(config.MONGO_HOST, config.MONGO_PORT)[config.MONGO_DB]

def extract_data(mysql_cursor):
    """Given a cursor, extracts data from MySQL dataset and returns all the tables with their data"""
    topics = execute_mysql_query('select * from topics', mysql_cursor, 'fetchall')
    article_topics = execute_mysql_query('select * from article_topics', mysql_cursor, 'fetchall')
    ratings = execute_mysql_query('select * from ratings', mysql_cursor, 'fetchall')
    users = execute_mysql_query('select * from users', mysql_cursor, 'fetchall')
    articles = execute_mysql_query('select * from articles', mysql_cursor, 'fetchall')
    tables = (topics, article_topics, ratings, users, articles)
    return tables

def execute_mysql_query(sql, cursor, query_type):
    """exectues a given sql, pymysql cursor and type"""
    if query_type == "fetchall":
        cursor.execute(sql)
        return cursor.fetchall()
    elif query_type == "fetchone":
        cursor.execute(sql)
        return cursor.fetchall()
    else:
        pass

def transform_data(dataset, table):
    """Transforms the data to load it into mongoDB, returns a JSON object"""
    dataset_collection = []
    tmp_collection = {}
    if table == "topics":
        for item in dataset[0]:
            tmp_collection['_id'] = item[0]
            tmp_collection['topic'] = item[1]
            dataset_collection.append(copy.copy(tmp_collection))
        return dataset_collection
    elif table == "users":
        for item in dataset[3]:
            tmp_collection['_id'] = item[0]
            tmp_collection['age'] = item[1]
            tmp_collection['topic'] = item[2]
            tmp_collection['occupatein'] = item[3]
            tmp_collection['zip_code'] = item[4]
            dataset_collection.append(copy.copy(tmp_collection))
        return dataset_collection
    elif table == "articles":
        for item in dataset[4]:
            tmp_collection['_id'] = item[0]
            tmp_collection['title'] = item[1]
            tmp_collection['release_date'] = item[2]
            tmp_collection['body'] = item[3]
            tmp_collection['URL'] = item[4]
            # embedding article topics
            article_topics_collection = []
            for article_topics_item in dataset[1]:
                if article_topics_item[0] == tmp_collection['_id']:
                    article_topics_collection.append(copy.copy(article_topics_item[1]))
            tmp_collection['topics'] = article_topics_collection
            # embedding ratings
            ratings_collection = []
            for ratings_item in dataset[2]:
                if ratings_item[1] == tmp_collection['_id']:
                    tmp_ratings_collection = {}
                    tmp_ratings_collection['user_id'] = ratings_item[0]
                    tmp_ratings_collection['rating'] = ratings_item[2]
                    tmp_ratings_collection['hirschindex'] = ratings_item[3]
                    ratings_collection.append(copy.copy(tmp_ratings_collection))
            tmp_collection['ratings'] = ratings_collection
            dataset_collection.append(copy.copy(tmp_collection))
        return dataset_collection

def load_data(mongo_collection, dataset_collection):
    """Loads the data into mongoDB and returns the results"""
    if RESET_MONGO_COLLECTIONS_ON_UPDATE:
        mongo_collection.delete_many({})
    return mongo_collection.insert_many(dataset_collection)

def main():
    """main method starts a pipeline, extracts data, transforms it and loads it into a mongo client"""
    if PRINT_INFO:
        print('Starting data pipeline')
        print('Initialising MySQL connection')
    mysql = initalise_mysql()
    
    if PRINT_INFO:
        print('MySQL connection Completed')
        print('Starting data pipeline stage 1 : Extracting data from MySQL')
    mysql_cursor = mysql.cursor()
    mysql_data = extract_data(mysql_cursor)
    
    if PRINT_INFO:
        print('Stage 1 completed! Data successfully extracted from MySQL')
        print('Starting data pipeline stage 2: Transforming data from MySQL for MongoDB')
        print('Transforming topics dataset')
    genres_collection = transform_data(mysql_data, "topics")
    
    if PRINT_INFO:
        print('Successfully transformed topics dataset')
        print('Transforming users dataset')
    users_collection = transform_data(mysql_data, "users")
    
    if PRINT_INFO:
        print('Successfully transformed users dataset')
        print('Transforming articles dataset')
    articles_collection = transform_data(mysql_data, "articles")
    
    if PRINT_INFO:
        print('Successfully transformed users dataset')
        print('Stage 2 completed! Data successfully transformed')
        print('Intialising MongoDB connection')
    mongo = initalise_mongo()
    
    if PRINT_INFO:
        print('MongoDB connection Completed')
        print('Starting data pipeline stage 3: Loading data into MongoDB')
    result = load_data(mongo['topics'], topics_collection)
    
    if PRINT_RESULTS:
        print('Successfully loaded topics')
        print(result)
    result = load_data(mongo['users'], users_collection)
    
    if PRINT_RESULTS:
        print('Successfully loaded users')
        print(result)
    result = load_data(mongo['articles'], articles_collection)
    
    if PRINT_RESULTS:
        print('Successfully loaded users')
        print(result)
    
    if PRINT_INFO:
        print('Stage 3 completed! Data successfully loaded')
        print('Closing MySQL connection')
    mysql.close()
    
    if PRINT_INFO:
        print('MySQL connection closed successfully')
        print('Ending data pipeline')

if __name__ == "__main__":
    main()
