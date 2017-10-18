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



---

# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify). 
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)  
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

I am submitting the writeup as markdown and You're reading it!

### Perception pipeline

Below is overall flowchart for perception pipeline:

![alt text][flowchart]

#### 1. Pipeline for filtering and RANSAC plane fitting implemented.

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

Clustering after performing above steps:

| Test world 1                     | Test world 2                     | Test world 3                     |
| :------------------------------: | :-------------------------------:| :-------------------------------:|
| ![alt text][test1_clustering_cv] | ![alt text][test2_clustering_cv] | ![alt text][test3_clustering_cv] |

| Test world 1                  | Test world 2                  | Test world 3                  |
| :---------------------------: | :----------------------------:| :----------------------------:|
| ![alt text][test1_clustering] | ![alt text][test2_clustering] | ![alt text][test3_clustering] |

#### 3. Features extracted and SVM trained.  Object recognition implemented.

Confusion Matrix:

![alt text][confusion_matrix]

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

Object recognition after performing above steps:

| Test world 1                      | Test world 2                      | Test world 3                      |
| :-------------------------------: | :--------------------------------:| :--------------------------------:|
| ![alt text][test1_objectrecog_cv] | ![alt text][test2_objectrecog_cv] | ![alt text][test3_objectrecog_cv] |

| Test world 1                         | Test world 2                         | Test world 3                         |
| :----------------------------------: | :-----------------------------------:| :-----------------------------------:|
| ![alt text][test1_objectrecognition] | ![alt text][test2_objectrecognition] | ![alt text][test3_objectrecognition] |

Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.  



