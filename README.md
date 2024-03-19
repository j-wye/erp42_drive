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
### Expression of Relation between Pixel Coordinate and World Coordinate
*Precondition : Camera is mounted on the top of the front of the vehicle, in the **center of the car**, **looking down***

**<span style="color:indigo"> Pixel Coordinate :</span> $\color{indigo}(cx, cy)$**
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

**<span style="color:indigo"> World Coordinate :</span> $\color{indigo}(WX, WY)$**

Camera height at World : $CH_W = (0, 0, h_W)$

Camera $NP_C$, $FP_C$과 매칭되는 월드 좌표계에서의 좌표 : $NP_W$, $FP_W$
- Nearest Point at World : $NP_W = (np_{WX},\,0,\,0)$
- Farthest Point at World : $FP_W = (fp_{WX},\,0,\,0)$
- Detected Point at World : $DP_W = (dp_{WX}\,,\,dp_{WY}, 0)$

Expression of Relation between $dp_{cy}$ and $dp_{WX}$
$$\color{blue}dp_{WX} = \frac{(ch\,-\,dp_{cy})}{ch}\times (fp_{WX} - np_{WX}) + np_{WX}$$
- **픽셀 좌표계에서의 y좌표의 위치가 월드 좌표계에서의 X좌표의 위치와 일대일 대응이라는 생각**

Right Triangle between three points : $Origin, CH_W,\,DP_W|_{y=0}$

$$\overrightarrow{CH_W\,DP_W|_{y=0}} = (dp_{WX},\,0\,-h_W)$$

$$\overline{CH_W\,DP_W|_{y=0}} = \sqrt{dp_{WX}^2\,+\,h_W^2}$$

Right Triangle between three points : $Origin,\,DP_W|_{y=0},\,DP_W$

$$\overline{DP_W\,DP_W|_{y=0}} = \tan\alpha\times\overline{CH_W\,DP_W|_{y=0}}$$

$$\color{blue}dp_{WY} = \tan\alpha\times\sqrt{dp_{WX}^2\,+\,h_W^2}$$

***최종적으로 차량이 존재하는 평면은 월드 좌표계에서의 XY평면이므로 steering angle을 세타라 하자.***
$$\tan\theta = \frac{dp_{WY}}{dp_{WX}}$$
$$\color{red}\theta = \arctan{\frac{dp_{WY}}{dp_{WX}}}$$

따라서, $f(dp_{cx},\,dp_{cy},\,cw,\,ch,\,fp_{WX},\,np_{WX},\,h_W)\,=\,\theta$

But, input은 카메라에서 detect되는 점인 $dp_{cx},\,dp_{cy}$ 밖에 없다.

$\therefore cw,\,ch,\,fp_{WX},\,np_{WX},\,h_W$ 이 5개의 변수는 상수값이어야 $\theta$ 를 구할 수 있다.

### 3. Additional expression
Camera Model : Intel realsense D435 
```
HFOV(Horizontal Field of View) : 69.4
VFOV(Vertical Field of View) : 42.5
```
*$\phi$ : 카메라를 아래로 기울인 각도*

if $\phi$ > 21.25 $\degree$ : 카메라가 아래로 VFOV의 절반인 21.25보다 크다면 카메라의 frame에는 바닥만이 잡히게 된다.

1. By VFOV
- 카메라 VFOV는 Z축을 기준으로 각도가 $(-\frac{VFOV}{2}, \frac{VFOV}{2})$ 였고, $\phi$ 만큼 아래로 회전했으므로, $(\phi\,-\,\frac{VFOV}{2},\,\phi\,+\,\frac{VFOV}{2})$ 가 된다.
- 카메라 FOV의 아랫변과 카메라를 포함하는 평면을 C1, 카메라 FOV의 윗변과 카메라를 포함하는 평면을 C2라 하자.
- Plane $C_1$의 normal vector $\overrightarrow{N_1}\,=\,(\tan(\phi\,+\,\frac{VFOV}{2}),\,0,\,1)$, Plane $C_2$의 normal vector $\overrightarrow{N_2}\,=\,(\tan(\phi\,-\,\frac{VFOV}{2}),\,0,\,1)$ 가 된다.

2. By HFOV
- 카메라의 FOV의 왼쪽 변을 $L_1$ 오른쪽 변을 $L_2$라 하자.
- $L_1$과 $L_2$은 XZ평면에서 봤을 때, X축에 수직인 선분이다.
- 이를 $\phi$만큼 회전시키
- 카메라의 HFOV는 Y축을 기준으로 각도가 $(-\frac{HFOV}{2}, \frac{HFOV}{2})$ 였고, $\phi$ 만큼 아래로 회전했다.
- 