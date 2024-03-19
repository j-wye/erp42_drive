# 한국교통 안전공단 주관 자율주행 경진대회 (2023)

The Mission : **<U>Drive a track made with rubber cones without lanes</U>**

## 1. Vision
### 1. Object Detection for Rubber Cone
- 1. Download Rubber Cone dataset
    ```bash
    # Bounding Boxes
    wget http://fsoco.cs.uni-freiburg.de/datasets/fsoco_bounding_boxes_train.zip

    # Panoptic Segmentation
    wget http://fsoco.cs.uni-freiburg.de/datasets/fsoco_segmentation_train.zip
    ```

- 2. At Roboflow, decide which YOLO version to use (e.g. YOLOv8). Then download dataset which suitable for that version

- 3. Train with selected YOLO version

- 4. Connect and detect with real-time

### 2. How to use detected Rubber Cone
- 1. All of rubber cones detect which caughted in the camera frame

- 2. Split the camera frame in half, left and right

- 3. Because of using a monocular camera, I detect the largest object in each half of the frame
    - Keypoint : As I can't know depth value, I assume that the largest detected object is the closest one

- 4. For each frame, I extracted the camera coordinates of the center of the lowermost detected object

- 5. Then I extracted the two centers of the coordinates extracted from each frame and set their centroid as the **goal point**.

## 2. Control Logic
- 