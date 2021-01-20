---
layout: single
title: "Social Media Caption Generator "
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "Machine Learning,Data Collection,Deep Learning,Image Processing,Multi CLass Classification,Natural Language Processing,Python"
mathjax: "true"
---
## Description:
This project has the capability of being used by social networking platforms or other image-dependent platforms as a “Automatic Social Media Caption Generator”. Social media platforms like Instagram, Facebook, Twitter, Pinterest are highly dependent on how the image descriptions are set up with a good caption and hashtags. Our model analyzes the objects in the image and generates categories that can be used as “hashtags” and further recommends captions to the user as they upload images into the application. 

## Technologies Used:
- Predictive Modeling
- Deep Learning
- Image Processing
- MultiClass Text Classification
- Web Scrapping

## Environment
Python (NumPy,Pandas,Tkinter,Scikitlearn,NLTK,TensorFlow, Keras),YouTube API

## Flow Diagram
<u>Phase 1: Generating Image description</u><br>
In this phase, we implemented two sequential models using pre-trained models in Keras with Tensorflow backend. The first model is responsible to extract the features from images for which we are using the Residual Network Model. The second model is for creating word embeddings after performing Natural Language processing on the image descriptions in the trained dataset.The Flicker30k dataset was used in order to train the two sequential models.
<br>
![picture3](/images/Caption_Generator/Picture3.png)

<u> Phase 2: Predicting the Category of the Image Description </u><br>
In this phase, we used the output from phase 1 to predict the category of the image to classify the image description into 9 different categories adventure, art and music, food, history, manufacturing, nature, science and technology,sports,travel. As the labelled dataset wasn't openly available , a labelled dataset was created by collecting the data using Youtube API and labelling the youtube video descriptions based on different queries.

<br>
![picture2](/images/Caption_Generator/Picture2.png)

<u> Phase 3: Building the UI </u>
<br>
In this phase, a UI was built using TKinter to run the models that were built in phases 1 and 2. Post development of the UI, an executable file was created which can be used by anyone to run this application easily. 
<br>
![picture1](/images/Caption_Generator/Picture1.png)


## DEMO EXECUTION 
Click below on the Video

<div align="center">
      <a href="https://youtu.be/pN-hK5NqQjk">
     <img 
      src="https://img.youtube.com/vi/pN-hK5NqQjk/0.jpg" 
      alt="Dashboard" 
      style="width:50%;">
      </a>
</div>
    


