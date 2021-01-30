---
layout: single
title: "Image Sorter Application"
#date: 2020-08-15
#tags: [data wrangling, data science, messy data]
classes: wide
excerpt: "Machine Learning,Transfer Learning,Image Processing, Python ,App Development"
mathjax: "true"
---
## Description
An image browsing application which allows an automatic sorting of images. The python app sorts the Dog Images based on the image position(sideways or upright) using Convolutional Neural Network and Transfer Learning.

![alt text](/images/Image_Sorter_App/App-UI-Screenshot.png)

Folder With Pictures : Inputs the folder path(Source) which includes the images to be sorted.<br>
Destination Folder 1 : Inputs the folder path where the sorted Upright images are saved.<br>
Destination Folder 2 : Inputs the folder path where the sorted Sideways images are saved.<br>

Upon clicking the Process,predictions for the images in the source directory are performed to identify whether the image is upright or sideways and required images are stored in their respective directories either Destination Folder 1 if upright or Destination Folder 2 if sideways.

## Predict the position of the image(Upright vs sideways)
Implemented a machine learning model by fine tuning the ResNet50,a convolutional neural network that is 50 layers deep. Since the training data available was less implementing Transfer learning and fine-tuning to train the model and further applying data Augmentation to images was faster and gave good results. To implement Transfer learning, we removed the last predicting layer of the pre-trained ResNet50 model and replace them with our own predicting layers. Weights of ResNet50 pre-trained model is used as feature extractor. Weights of the pre-trained model are frozen and are not updated during the training.

## Application
Application was created using the Python's Tkinter framework thus creating a standalone application which can be further used by the photograhers in to sort their daily/weekly or monthly clicked images.

## Technology Used
 - Convolutional Neural Network 
 - Transfer Learning        

## Environment 
 - Python (NumPy,Pandas,Tkinter,TensorFlow, Keras)

## DEMO EXECUTION
**Click below on the Video**<br>
<div align="center">
      <a href="https://youtu.be/Naf5__i5vDU">
     <img 
      src="https://img.youtube.com/vi/Naf5__i5vDU/0.jpg" 
      alt="Everything Is AWESOME" 
      style="width:50%;">
      </a>
    </div>


