# 한국교통 안전공단 주관 자율주행 경진대회 (2023)

The Mission : **<U>Drive a track made with rubber cones without lanes</U>**

## 1. Vision
### 1. Object Detection for Rubber Cone
- Download Rubber Cone dataset
    ```bash
    # Bounding Boxes
    wget http://fsoco.cs.uni-freiburg.de/datasets/fsoco_bounding_boxes_train.zip

    # Panoptic Segmentation
    wget http://fsoco.cs.uni-freiburg.de/datasets/fsoco_segmentation_train.zip
    ```

- At Roboflow, decide which YOLO version to use (e.g. YOLOv8). Then download dataset which suitable for that version

- Train with selected YOLO version

- Connect and detect with real-time

### 2. How to use detected Rubber Cone
- All of rubber cones detect which caughted in the camera frame

- Split the camera frame in half, left and right

- Because of using a monocular camera, I detect the largest object in each half of the frame
    - Keypoint : As I can't know depth value, I assume that the largest detected object is the closest one

- For each frame, I extracted the camera coordinates of the center of the lowermost detected object

- Then I extracted the two centers of the coordinates extracted from each frame and set their centroid as the **goal point**.

## 2. Control Logic
### 1. Expression of Relation between Pixel Coordinate and World Coordinate
Precondition : Camera is mounted on the top of the front of the vehicle, in the **center of the car**, **looking down**
- 픽셀 좌표계를 (cx, cy)라 하자
    ```
    cx^max^ = width
    cy_max = heiht
    cx_min = 0
    cy_min = 0
    ```
- 