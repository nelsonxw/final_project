# CNN Model Fine-tuning and Flower Classifications

### Problem Statement:  
What is an effective way to satisfy the curiosity of a five-year old kid?  When walking in a beautiful botanical garden with blooming flowers, how to utilize machine learning to help a human being learn the names of different flowers?

### Objective:

+ Compare 10 pre-trained Keras CNN models on their effectiveness in classifying flower types.
+ Pick the best pre-trained model and fine-tune it to improve its prediction accuracy and achieve better reductions in loss.
+ Create a simple web app where uses can upload pictures and find out the name of flowers

### Dataset:

+ Scrapped pictures of five species (daisy, dandelion, rose, sun flower and tulip) from ImageNet website for training and validation
+ Downloaded flower picture (same five species) dataset from Kaggle for testing

### Tools used:
Python, Pandas, HTML, CSS, JavaScript, TensorFlow, Keras, Scikit-learn, Matplotlib, Flask, Heroku

### Experiment Steps:
1. Downloaded initial data from Kaggle (https://www.kaggle.com/alxmamaev/flowers-recognition/home) and split it into training dataset and test dataset.  With the test dataset, directly applied the pre-trained CNN models to compare and pick which one works best in predicting the flower types.  
[code link](/1-pretrained_models_comparison.ipynb)  
    + The result is very surprising.  All 10 modesl failed to predict dandelion, rose, sun flower and tulip, and scored 0 accuracy with these flowers.  The models scored between 75% and 85% when predicitng daisy.  
        <img src="/screen%20shots/pre-trained_model_comparison.PNG" width="400">
    + Used one of the models (Xception) to verify the predicted flower names under each category and it totally mis-predicted the names.  
[code link](/2-verification_Xception.ipynb)  

2. Since all 10 models were trained with ImageNet pictures, decided to scrap pictures from ImageNet website and downloaded data for further validations.
    + Installed library from https://github.com/tzutalin/ImageNet_Utils, tweaked the codes to work for Python 3.  
    [code link](https://github.com/nelsonxw/Modified_ImageNet_Utils/tree/71287d9543cf939a62889c191aab7aea46876434)
    + Used the ID of flowers to download pictures
    + Used ImageNet images to predict flower names, and got same results.  
    [code link](/3-verification_ImageNet.ipynb)

3. At this point, the only conclusion I can get is that the pre-trained models were traied to do other classifications, and cannot be used directly to predict the flower species I have.  However, these models can be fine-tuned and trained with my own data to classify the five different flower names.  Since InceptionResNetV2 scored the highest in step 1, I picked it for initial fine-tuning.
    + Because it is very time consuming to run each epoch on my PC, I initially selected small training dataset with 50 pictures for each species to get a quick feel of the model.
    + When pre-processing the images, I used data augmentation function (rotate, shift, flip) to increase the amount of training data and reduce overfit.
    + I tweaked different parameters (# of layers to train, # of nodes in the new layer, drop out rate, batch size, optimizer, learning rate, # of epochs) and tried to improve prediction accuracy and reduce loss.  The accuracy kept going up and loss kept going down in the training dataset with each epoch, however, the best result I could get from the validation dataset was only about 55%, with clear evidence of overfitting in the model.  
<img src="/screen%20shots/InceptionResNetV2_result1.PNG" width="300"> <img src="/screen%20shots/InceptionResNetV2_result2.PNG" width="300">
4. After some research, I learned that reducing the complexity of the network may reduce overfitting problems.  With that in mind, I compared the 10 models in terms of how many layers do they have (more layers indicate more complexity).  It turned out VGG16 has the least amount of layers, so I switched to fine tune VGG16 model.
    + Complexity of each model  
        <img src="/screen%20shots/compare_model_complexity.PNG" width="400">  
        
        [code link](/5-select_simple_model.ipynb)
    + Improved accuracy (75%) and reduction in loss  
        <img src="/screen%20shots/VGG16_initial_result1.PNG" width="300"> <img src="/screen%20shots/VGG16_initial_result2.PNG" width="300">  
        [code link](/6-finetune_model_VGG16.ipynb)
5. I tried to increase the learning rate by 3x in the hope of finding the global optimum quicker, but it ended up with more overfitting.  
    <img src="/screen%20shots/VGG16_worse_result1.PNG" width="300"> <img src="/screen%20shots/VGG16_worse_result2.PNG" width="300">  
    [code link](/7-finetune_model_VGG16_2.ipynb)
6. I kept the learning rate at 0.00001 (use small learning rate for fine tuning, otherwise it could over shoot and never get to the minimum gradient descent point).  In order to further reduce overfitting, I increased the amount of training data from 50 pictures per species to 450 pictures per species.  The prediction accuracy improved to 88% with further reduction in loss.  
    <img src="/screen%20shots/VGG16_better_result1.PNG" width="300"> <img src="/screen%20shots/VGG16_better_result2.PNG" width="300">  
    [code link](/8-finetune_model_VGG16_3.ipynb)
7. I tested with differnet optimizer functions and settled on using RMSprop.  I kept 100 pictures per species as validation dataset, and used all the remaining pictures from ImageNet (over 1000 pictures per species) as the training data.  The prediction accuracy improved to 95% with the loss in the range of 0.2 to 0.3.  Created confusion matrix and it showed good precision and recall measurements for all flowers (daisy slightly lower than others).  
    <img src="/screen%20shots/VGG16_final_result1.PNG" width="300"> <img src="/screen%20shots/VGG16_final_result2.PNG" width="300">  
    <img src="/screen%20shots/VGG16_final_confusion_matrix.PNG" width="400">  
    [code link](/9-finetune_model_VGG16_final.ipynb)
8. I saved the fine-tuned model and used it to predict flower names based on the validation data.  I was trying to verify that the model can truly deliver the high prediction accuracy.  
    + Initially I got really poor results, accuracy under 40%.  Being shocked, I knew there must be something wrong with my codes.  After careful examination, I realized that I was using the pre-processing function imported from VGG16 model.  Since it has been fine turned with my own pre-processing steps, the inconsistency in the pre-processing has caused my model to return incorrect predictions.
        ```python
        x = preprocess_input(x)
        ```
    + I corrected the code and used the same pre-processing steps I had when I fine-tuned VGG16 model.  The prediction results are aligned with expectations now.
        ```python
        x = x / 255
        ```  
        <img src="/screen%20shots/prediction_ImageNet_pictures.PNG" width="200">  
        
        [code link](/10-test_my_model_ImageNet_images.ipynb)
9. Used the saved model to predict flower names based on Kaggle's test dataset.  The prediction accuracy decreased across the species compared to the results from validation data.  This is expected result because the model tried hard to fit to the validation data when I was tweaking parameters in the fine-tuning process.  When predicting new data that the model had never seen before, its performance will likely deteriorate to some degree.  In addition, the quality of the test data may not be as good as the training / validation data, resulting in lower accuracy.  However, the overall prediction accuracy is still much better than the initial result in step 1.  
    <img src="/screen%20shots/prediction_Kaggle_pictures.PNG" width="200">  
    
    [code link](/11-test_my_model_Kaggle_images.ipynb)
    
10.  Used Flask to serve a web application where users can select a picture from local folders and upload to the web.  The fine-tuned model will predict the flower name, and return the wikipedia search result for the predicted flower.  
        + Tried to use Heroku to host the web.  But the saved model is over 100MB in size, and found out Heroku doesn't support git lfs (for large files).  Tried another approach by setting up a route (/model_upload) to allow users upload the model before uploading the pictures.  In this way, the saved model file doesn't have to be moved to Heroku directory.  Found out this approach will only work if the upload speed is fast: the model upload can be completed within 30 seconds.  Heroku has a timeout limitation of 30 seconds, so if the upload takes longer than that, the connection will drop.  
            <img src="/screen%20shots/web_application.PNG" width="800">
### Lessons Learned:
+ Start with a smaller set of data to get a quick feel of the model.
+ If you have a fast computer with GPU and enough time, use as much training data as you can and tweak the parameters as much as you can.
+ Reducing the complexity of the model or algorithm may help minimize overfitting issues.
+ Always validate the model results before celebrating for success.
+ It will be a good practice to split data into three parts: training data, validation data and test data.  As evidenced in this experiment, the results from validation data may be better than the actual performance it can generate in unseen test data.  The measurements from test data will be a more reliable indicator.
+ Recognize the limitations of supervised learning and classification algorithms.  If a model was not trained for a particular class, then it will fail to predict that particular class.  To truly satisfy the curiosity of a child and answer open ended questions (e.g. unlimited flower species), a supervised learning model may not be the best option.

