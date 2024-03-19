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

**<span style="color:red"> Pixel Coordinate :</span> $\color{red}(cx, cy)$**
- $cx_{max} = cw$ (camera width)
- $cy_{max} = ch$ (camera height)
- $cx_{min} = 0$
- $cy_{min} = 0$

<!-- At **$\color{red}cx = cw\,/\,2$** :  -->
- Nearest Point at Camera : $NP_C = (cw\,/\,2,\,ch)$
- Farthest Point at Camera : $FP_C = (cw\,/\,2,\,0)$
- Detected Point at Camera : $DP_C = (dp_{cx}\,,\,dp_{cy})$

At $cx = cw/2$, $cy = dp_{cy}$ :
- $NP_C$ $DP_C$ $DP_C|_{x = cw/2}$ 이 세점으로 만들어진 직각삼각형의 사이각을 $\alpha$
$$\tan\alpha = \frac{dp_{cx}\,-\,cw/2}{ch - dp_{cy}}$$

**<span style="color:red"> World Coordinate :</span> $\color{red}(WX, WY)$**

Camera height at World : $CH_W = (0, 0, h_W)$

Camera $NP_C$, $FP_C$과 매칭되는 월드 좌표계에서의 좌표 : $NP_W$, $FP_W$
- Nearest Point at World : $NP_W = (np_{WX},\,0,\,0)$
- Farthest Point at World : $FP_W = (fp_{WX},\,0,\,0)$
- Detected Point at World : $DP_W = (dp_{WX}\,,\,dp_{WY}, 0)$

Expression of Relation between $dp_{cy}$ and $dp_{WX}$
$$dp_{WX} = \frac{(ch\,-\,dp_{cy})}{ch}\times (fp_{WX} - np_{WX}) + np_{WX}$$
- **픽셀 좌표계에서의 y좌표의 위치가 월드 좌표계에서의 X좌표의 위치와 일대일 대응이라는 생각**

Right Triangle between three points : $Origin, CH_W,\,DP_W|_{y=0}$

$\vec{CH_W\,DP_W|_{y=0}} = (dp_{WX},\,0\,-h_W)$

$\overline{CH_W\,DP_W|_{y=0}} = \sqrt{dp_{WX}^2\,+\,h_W^2}$


