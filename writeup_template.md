##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I started by reading data from files provided . I found out there are two classes of data whcih is present i.e.
vehicle (Car) data and non-vehicle data non-vehcicular data(notcar) . I also found number of labeled data for 
both the classes is almost if not exactly equal  . There were 8792 labeled car images and 8968 images for not-car images. 
In order to determine the HOG feature of an image I have defined a function get_hog_feature (taken from lesson) which
takes input as images , number of orientations (9 in this case , an optimum value defined in HOG documentation aslo) ,
pixel per cell = 8 and cell per block =2 .  This function uses a in built library function hog() which is part of skimage.feature
and returns a feature vector (Histogram of oriented gradients) . So we called this function and it returned a hog feature
vector. Then I displayed car and non car images along with HOG visulziation of the image (after converting the image to gray scale
in this case in  Cell #4 and 5 of ipython notebook. 

####2. Explain how you settled on your final choice of HOG parameters.

Apart from the details above when it comes to specifically for  HOG , I started with HOG number of orientations = 6 , but used 9 at the end (as it was recommended in documentation
that this gives best result)  . I used pix_per_cell = 8 and cell_per_block = 2 and used hog function whcih is part of 
skimage.feature import hog to find historgam oriented of gradient feature vector.  I also set visualization= True so that
I coould see output of car and non car features (as shown in Cell  # 4 and 5) . 

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the following parameters : 
color_space = 'YCrCb'  # RGB, HSV, LUV, HLS, YUV, YCrCb
orient = 9
pix_per_cell = 8
cell_per_block = 2
hog_channel = 'ALL'  # 0, 1, 2, "ALL"
spatial_size = (32, 32)
hist_bins = 32
spatial_feat = True
hist_feat = True
hog_feat = True

Also for running SVC , I used regularization parameter c = 0.00002 as follows : 
svc= LinearSVC(C=0.00002)  (You can see full code in 17th Code Cell)


I extracted car features and non car features for color_space 'YCrCb' (I found out this color space the min false
positives after lot of iterations running images for RGB , HSV and LUV ) . for hog i used orientation = 9 ( as recommended
in HOG documentation) , pix_per_cell =8 and cells_per_bloc = 2 . I started with hog_channel = 0 but finally i settled
with hig_channel ='ALL'  w. These parameters gave me least false positives (after trying many iterations over differen
combination). 

After extracting car and non car features vector (each of this feature vector was a list of color space features , 
spatial binning , histogram and HOG features), I vertical stacked car and noncar features and then normalized this
feature vector using StandardScaler . After that I splitted the data into 90-10 ratio was train-test purpose using 
 sklearn.model_selection's train_test_split method .I then ran a Linear Support Vector Classifier on a feature vector of length
 8460 and got test accuracy of 99.16% .



###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

To implement this I used single function find_cars that's able to both extract features and make predictions.
This function extract hog features once and which can be sub-sampled to get all of its overlaying window . 
Each window is defined by a scaling factor and  overlap of each window is in terms of the cell distance . 
I earlier ran this function for [1,1.5,2] but finally got good results with[1.5,2]  i.e.
for multiple scale factor so as to generate multiple-scaled search windows which
is a more robust and efficient way .



####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

My examples are listed in Cell # 18  . As explained above to optimize it i used YCrCb , orient=9 and two scales of 1.5 and 2
in find_car function (whcih I had taken from lesson) . I derived at these hyper parameters after lot of trials and 
found out these to be giving least false positives. 

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

My final video output code is in cell # 19 and 20 and output video I have attached with the name "P5_project_output.mp4'



####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.
From the positive detections I created a heatmap and then thresholded that map
to identify vehicle positionsI then used `scipy.ndimage.measurements.label()`
to identify individual blobs in the heatmap.  I then assumed each blob
corresponded to a vehicle.  I constructed bounding boxes to cover
the area of each blob detected.Also  ,I identified only those vehicles
which came in 10 out of 14 frames . Code for it is present in Cell # 19


###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Even after trying many iterations and tuning and changing parameters there were
still quite a few fasle positives which were crept into the video . Since the time
is less and implementing and tuning this took long time , I would have wanted
to check deep learning approach using Keras to get this identified and see 
if there are any false positives , because at the end we are doing "image classification"
and keras/deep learning might be an approach I would have liked to check 
its output . 
