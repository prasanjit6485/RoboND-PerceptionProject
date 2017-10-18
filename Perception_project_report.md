## Project: Perception Pick & Place

[//]: # (Image References)

[flowchart]: ./images/PerceptionPipeline.jpg

[test1_extractinliers_cv]: ./images/test1_extractinliers_cameraview.png
[test2_extractinliers_cv]: ./images/test2_extractinliers_cameraview.png
[test3_extractinliers_cv]: ./images/test3_extractinliers_cameraview.png
[test1_extractinliers]: ./images/test1_extractinliers.png
[test2_extractinliers]: ./images/test2_extractinliers.png
[test3_extractinliers]: ./images/test3_extractinliers.png
[test1_extractoutliers]: ./images/test1_extractoutliers.png
[test2_extractoutliers]: ./images/test2_extractoutliers.png
[test3_extractoutliers]: ./images/test3_extractoutliers.png

[test1_clustering_cv]: ./images/test1_clustering_cameraview.png
[test2_clustering_cv]: ./images/test2_clustering_cameraview.png
[test3_clustering_cv]: ./images/test3_clustering_cameraview.png
[test1_clustering]: ./images/test1_clustering.png
[test2_clustering]: ./images/test2_clustering.png
[test3_clustering]: ./images/test3_clustering.png

[confusion_matrix]: ./images/confusion_matrix.png
[test1_objectrecog_cv]: ./images/test1_objectrecognition_cameraview.png
[test2_objectrecog_cv]: ./images/test2_objectrecognition_cameraview.png
[test3_objectrecog_cv]: ./images/test3_objectrecognition_cameraview.png
[test1_objectrecognition]: ./images/test1_objectrecognition.png
[test2_objectrecognition]: ./images/test2_objectrecognition.png
[test3_objectrecognition]: ./images/test3_objectrecognition.png

[output]: ./images/output.png

![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

I am submitting the writeup as markdown and You're reading it! 

P.S.: I have not performed the extra challenges.

### Perception pipeline

Below is overall flowchart for perception pipeline:

![alt text][flowchart]

#### 1. Pipeline for filtering and RANSAC plane fitting implemented.

Step1: We perform filtering and RANSAC in PCL, convert ROS data to PCL data
Step2: Apply PCL based VoxelGrid Downsampling Filter to downsample the data by considering a reasonable voxel (leaf) size. LEAF size selected for this dataset along each axis is 0.01.
Step3: Since `/pr2/world/points` topic contains noisy point cloud data, apply PCL based Outlier Removal Filter to remove external noise.
Step4: Apply PCL based Pass Through Filtering to remove useless data from the point cloud. First in `z` direction where axis_min = 0.6, axis_max = 1.1 were selected and then in `y` direction axis_min = -0.5, axis_max = 0.5 were selected.
Step5: Apply PCL based RANSAC plane fitting algorithm to extract inliers and outliers by considering max distance for a point to be considered fitting the model. MAX distance selected for this dataset is 0.01. 

Extract inliers (Objects) after performing above steps:

| Test world 1                         | Test world 2                         | Test world 3                         |
| :----------------------------------: | :-----------------------------------:| :-----------------------------------:|
| ![alt text][test1_extractinliers_cv] | ![alt text][test2_extractinliers_cv] | ![alt text][test2_extractinliers_cv] |

| Test world 1                      | Test world 2                      | Test world 3                      |
| :-------------------------------: | :--------------------------------:| :--------------------------------:|
| ![alt text][test1_extractinliers] | ![alt text][test2_extractinliers] | ![alt text][test3_extractinliers] |

Extract outliers (Table) after performing above steps:

| Test world 1                       | Test world 2                       | Test world 3                       |
| :--------------------------------: | :---------------------------------:| :---------------------------------:|
| ![alt text][test1_extractoutliers] | ![alt text][test2_extractoutliers] | ![alt text][test3_extractoutliers] |


#### 2. Pipeline including clustering for segmentation implemented.  

Step6: From previous pipeline, extract inliers (object) and apply PCL based EuclideanClusterExtraction() to perform a DBSCAN cluster search on your 3D point cloud.
Step7: After experimenting with the datasets, tolerances for distance threshold is 0.015 as well as minimum and maximum cluster size (in points) is selected as 20 and 2000.
Step8: After clustering (segmentation), label each object by assigning color to each object.
Step9: Now convert the PCL data to ROS data and publish inliers, outliers and clusters messages.

Clustering after performing above steps:

| Test world 1                     | Test world 2                     | Test world 3                     |
| :------------------------------: | :-------------------------------:| :-------------------------------:|
| ![alt text][test1_clustering_cv] | ![alt text][test2_clustering_cv] | ![alt text][test3_clustering_cv] |

| Test world 1                  | Test world 2                  | Test world 3                  |
| :---------------------------: | :----------------------------:| :----------------------------:|
| ![alt text][test1_clustering] | ![alt text][test2_clustering] | ![alt text][test3_clustering] |

#### 3. Features extracted and SVM trained.  Object recognition implemented.

Extract features like color and surface normals and train an SVM model on objects under pick_list_3.yaml file using sensor_stick node. The extracted features and trained model is saved under train_datasets folder. Below are some experiments conducted to improve the overall SVM accuracy for objects under pick_list_3.yaml file:

| Color (nbins/bins_range) | Surface normal (nbins/bins_range) | Number of times same object trained | SVM (%)  |
| :----------------------: | :--------------------------------:| :----------------------------------:| :------: |
| 32/(0,256)               | 128 (-1,1)                        | 50                                  | 95.5%    |
| 64/(0,256)               | 64 (-1,1)                         | 100                                 | 97.5%    |
| 64/(0,256)               | 128 (-1,1)                        | 100                                 | 98.5%    |

Below is the Confusion Matrix for the last experiment from above table:

![alt text][confusion_matrix]

Step10: For each segmented object, extract color and surface normal features and feed the features to SVM prediction model.
Step10: Finally recognized the object using saved trained SVM model and label each object based on prediction.

Object recognition after performing above steps:

| Test world 1                      | Test world 2                      | Test world 3                      |
| :-------------------------------: | :--------------------------------:| :--------------------------------:|
| ![alt text][test1_objectrecog_cv] | ![alt text][test2_objectrecog_cv] | ![alt text][test3_objectrecog_cv] |

| Test world 1                         | Test world 2                         | Test world 3                         |
| :----------------------------------: | :-----------------------------------:| :-----------------------------------:|
| ![alt text][test1_objectrecognition] | ![alt text][test2_objectrecognition] | ![alt text][test3_objectrecognition] |

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

From the perception pipeline, Performed object recognition and posted all the relevant screenshots for all three tabletop setups. I was able to correctly identified 100% of objects from pick_list_1.yaml for test1.world, 100% of items from pick_list_2.yaml for test2.world and 100% of items from pick_list_3.yaml in test3.world. 

![alt text][output]

Output yaml files are under output_yaml folder for each tabletop setups.

- [Test world 1](https://github.com/prasanjit6485/RoboND-PerceptionProject/blob/master/output_yaml/output_1.yaml)
- [Test world 2](https://github.com/prasanjit6485/RoboND-PerceptionProject/blob/master/output_yaml/output_2.yaml)
- [Test world 3](https://github.com/prasanjit6485/RoboND-PerceptionProject/blob/master/output_yaml/output_3.yaml)

Following are my conclusion where Perception project might fail:
1) Robot pr2 won't recognize object if the object is completely occluded.
2) Robot pr2 might not recognize object if the object is very noisy.
3) Robot pr2 might fail if the vision is not properly aligned to the reigion of interest.
4) Robot pr2 won't recognize object of same size, shape and color other than present in pick_list_3.yaml file. 
5) Robot pr2 might fail to recognize if the robot is not stable.

Following are my conclusion where I can improve my Perception pipeline:
1) Need to perform extra challenges to test my Perception pipeline.





