---
layout: single
title: "Streaming Data Analysis Pipeline - Twitter Data"
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "ETL Data pipeline,Twitter Data Analysis,Natural Language Processing,Sentiment Analysis,Dashboard,Python"
mathjax: "true"
---
Objective of this project is to implement a real time streaming data pipeline in order to 
- Analyse trending User Mentions, Hashtags 
- Most retweeted Tweets 
- Sentiment of the Tweets using Twitter data.

This real-time end-to-end Twitter monitoring system is designed for the enterprises to evaluate Twitter data.As we know twitter is a great place for the real-time high-throughput data source with 6000 tweets per seconds on average, we could use it to uncover the breaking news stories, identify industry-wide trends, and take actions on time. In practice, keep tracking of all relevant Twitter content in real-time,understand the users sentiment regarding a topic,trending user tweets.

## Technology Used
- Natural Language Processing 
- Real Time Streaming ETL 
- Predictive Modeling
- Sentiment Analysis
- Neural Networks

## Sentiment Analysis 
Before training the model all the twitter tweets data was cleaned to remove emoticons, puncuations, usernames, retweet keywords "RT" and further keeping only data in english language. Implemented a predictive model to classify the tweets as a positive sentiment or a negative sentiment using pre trained Google provided News corpus (3 billion running words) word vector model (3 million 300-dimension English word vectors) for generating the word embeddings. Further using the word embedding as the input data , 2 dense layer neural network and LightGBM were trained in order address the binary classification of sentiment. The neural network model outperformed with improved accuracy over LightGBM and thus was further implemeneted in the ETL dashboard.

## ETL Workflow
In order to allow the dashboard application to process records as they occur Apache Kafka was implemented as part of stream analysis. Elasticsearch for searching and filtering the tweet results and Kibana as it lets you see and interact with your data in realtime, as you’re gathering data and can be accessed directly from the browser.

## Enviornment
Python (scikitlearn, seaborn, pandas, numpy, matplotlib,Tensorflow,Keras),Jupyter Notebook,Apache Kafka, ElasticSearch , Kibana , Docker ,Twitter API

## DEMO EXECUTION:

**TWITTER DATA ANALYSIS FOR A MEDIA CHANNEL- Click below on the Video** <br>
<div align="center">
      <a href="https://youtu.be/wv96_7gRTG8">
     <img 
      src="https://img.youtube.com/vi/wv96_7gRTG8/0.jpg" 
      alt="Dashboard" 
      style="width:50%;">
      </a>
    </div>

## INSTALLATION

- Download Google Word Embedding:<br>
!wget -c "https://s3.amazonaws.com/dl4j-distribution/GoogleNews-vectors-negative300.bin.gz"

- STEPS :<br>
  **Create network:** <br>
    - $ docker network create kafka-network <br>
  **Spin up the local single-node cluster (will run in the background):**<br>
    - $ docker-compose -f docker-compose.kafka.yml up -d <br>
  **Check the cluster is up and running (wait for "started" to show up):**<br>
    - $ docker-compose -f docker-compose.kafka.yml logs -f broker | grep "started" <br>
  **Start the container **<br>
    - $ docker-compose up -d <br>
  **Create connection between Kafka and ElasticSearch:** <br>
curl -X POST -H "Content-Type: application/json" -d '
{
  "name": "test-connector",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "topic_test",
    "key.ignore": "true",
    "schema.ignore": "true",
    "connection.url": "http://elasticsearch:9200",
    "type.name": "test-type",
    "name": "elasticsearch-sink"
  }
}' localhost:8083/connectors

## References:
1. https://github.com/florimondmanca/kafka-fraud-detector <br>
2. https://medium.com/@raymasson/kafka-elasticsearch-connector-fa92a8e3b0bc <br>
3. https://stackoverflow.com/questions/48711455/how-do-i-create-a-dockerized-elasticsearch-index-using-a-python-script-running <br>
4. https://github.com/davidefiocco/dockerized-elasticsearch-indexer/blob/master/indexer/indexer.py <br>
