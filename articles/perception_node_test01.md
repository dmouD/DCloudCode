
# 机器狗视觉识别节点说明
## perception_node.py
## 1. 功能简介
本节点用于识别赛道中的橙色球、蓝色球、可乐瓶、足球等目标。

## 2. 参数列表
第一类：输入参数        
决定节点从哪里拿数据

第二类：输出参数  
决定节点把结果发到哪里 

第三类：算法参数  
决定识别规则和结果命名

| 变量类型 | 参数名称 | 作用 |  
| --- | --- | --- |
| 输入参数 | image_topic| 去哪个图像话题订阅 RGB 图 |
| 输出参数 | debug_image_topic | 调试图发那里 |
| 输出参数 | detection_topic | 全部检测结果发哪里 |
| 输出参数 | target_topic | 全部主目标放哪里 |
| 算法参数 | min_area | 忽略该数值以下的区域算噪声 |
| 算法参数 | orange_label | 橙球叫什么 |
| 算法参数 | blue_label | 蓝球叫什么 |

## 3.识别逻辑
/camera/image_raw  相机
   ↓   
感知节点识别球  
   ↓  
/vision/detections       发全部球结果  
/vision/target_point     发当前重点橙球  
/vision/debug_image      发画好框的调试图  
第一层:订阅层  
接收：
RGB 图  
深度图  
相机内参  
第二层：识别层  
在 RGB 图里做：  
- 颜色分割  
- 轮廓提取  
- 圆形筛选  
第三层：测距层  
- 在深度图里做：  
- 取中心深度  
- 反投影到 3D  
 第四层：输出层  
发布：  
- 所有球的位置  
- RViz marker  
- 当前主目标  

## 4.节点介绍
### **race_perception/perception_3d_node.py**
(1) 这个节点到底在干什么
它主要做 5 件事：
1. 订阅彩色图像  
2. 订阅深度图像  
3. 订阅相机内参  
4. 在彩色图中识别橙球和蓝球  
5. 用深度图把识别结果变成三维坐标，再发布出去
(2) 这个节点订阅的话题
- RGB图
```python 
self.rgb_sub = self.create_subscription(Image, self.rgb_topic, self.rgb_callback, 10) 
```
作用:拿到普通彩色画面  用来做颜色识别
- 深度图像
``` python 
self.depth_sub = self.create_subscription(Image, self.depth_topic, self.depth_callback, 10) 
```
作用：拿到每个像素对应的距离  用来算球离相机有多远
- 相机内参
```python
 self.cam_info_sub = self.create_subscription(CameraInfo, self.camera_info_topic, self.camera_info_callback, 10) 
 ```
作用：读出相机模型参数 fx fy cx cy  后面把像素坐标变成三维坐标时要用
(3)节点内部保存了什么
1. 相机内参  
```python 
  self.fx = None
  self.fy = None   
  self.cx0 = None 
   self.cy0 = None 
```
计算公式：
$$
像素点 (u, v) + 深度 d  ->  三维点 (x, y, z)
$$
2. 最新一帧深度图
```python
self.last_depth = None
self.last_depth_header = None
```
节点会先把最近收到的深度图存起来，等 RGB 图到了以后一起用
(4)主流程
收到RGB图
→ 转成OpenCV格式  
→ 转HSV颜色空间  
→ 分别提取橙色和蓝色区域  
→ 对每个颜色区域找轮廓  
→ 判断像不像球  
→ 找到球心像素坐标  
→ 去深度图取对应距离  
→ 反投影成3D坐标  
→ 发布结果  

## 5.运行逻辑
接收 RGB 图像  
→ 转 OpenCV 格式  
→ BGR 转 HSV  
→ 提取橙色掩膜和蓝色掩膜  
→ 去噪与平滑  
→ 提取候选球  
→ 过滤小区域和非球形区域  
→ 构造检测结果  
→ 选择当前优先橙球  
→ 发布检测结果  
→ 发布目标点  
→ 发布调试图  

## 6.依赖库
**Python 库**   
math：用于圆形度计算  
typing：用于类型标注  
cv2：OpenCV 图像处理  
numpy：矩阵和数组运算  
rclpy：ROS2 Python 接口  
cv_bridge：ROS 图像与 OpenCV 图像转换  
**ROS2 消息类型**   
sensor_msgs/msg/Image  
geometry_msgs/msg/PointStamped  
vision_msgs/msg/Detection2D  
vision_msgs/msg/Detection2DArray  
vision_msgs/msg/ObjectHypothesisWithPose  
std_msgs/msg/Header  


## 7.话题合集

### 订阅话题
/camera/image_raw 或参数指定的图像话题  
类型：  
sensor_msgs/msg/Image  
用途：接收相机原始 RGB 图像  

### 发布话题
/vision/detections  
类型：  
vision_msgs/msg/Detection2DArray  
用途：发布当前图像中所有检测到的橙球和蓝球结果  
内容包括：  
目标中心位置  
目标框大小  
类别 id  
置信度

/vision/target_point  
类型：geometry_msgs/msg/PointStamped  
用途：发布当前优先目标  
当前版本中：  
point.x 表示图像横向归一化偏差  
point.y 表示图像纵向归一化偏差  
point.z 表示目标面积  
这不是严格物理三维坐标，而是为了后续控制节点便于直接使用  

/vision/debug_image  
类型：  sensor_msgs/msg/Image  
用途：发布带有检测框、类别名称和当前目标标记的调试图像，供 rqt_image_view 或其他可视化工具查看。

# 8.代码实现
## 四个主要函数
- ### \_\_init\_\_ 初始化函数
- ### image_callback 图像回调函数
- ### clean_mask 掩膜清理函数
- ### extract_ball_candidates 提取橙球函数  
---------------------

clean_mask 掩膜清理函数
=
传入参数：self, mask: np.ndarray  
**传入一张掩膜图，返回清理后的掩膜图**  
**作用：清除噪点**  
------------------
*clean_mask内部*
```python
kernel = np.ones((5, 5), np.uint8)
mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)
mask = cv2.GaussianBlur(mask, (5, 5), 0)
```
定义一个 5×5 的卷积核，用于形态学操作  
开运算：先腐蚀，再膨胀，去掉小白点噪声  
闭运算：先膨胀，再腐蚀，填补小黑洞，让球区域更完整   
高斯模糊：让边界更平滑    

\_\_init\_\_ 初始化函数
=
```python
declare_parameter()
#先在节点里注册一个“可配置变量”，以后这个变量就能从 launch 文件、命令行或 YAML 配置里传进来。

self.declare_parameter('image_topic', '/camera/image_raw')  
self.declare_parameter('debug_image_topic', '/vision/debug_image') 
self.declare_parameter('detection_topic', '/vision/detections')
self.declare_parameter('target_topic', '/vision/target_point')
self.declare_parameter('min_area', 150)
self.declare_parameter('orange_label', 'orange_ball')
self.declare_parameter('blue_label', 'blue_ball')
```
### 声明节点参数  
- image_topic  
指定这个节点要去订阅哪一个 RGB 图像话题 默认普通彩色相机图像是从 /camera/image_raw 发出来的

- debug_image_topic  
指定这个节点把调试后的可视化图像发布到哪个话题 默认/vision/debug_image (调试图像为节点处理之后的图像)

- detection_topic  
指定这个节点把检测结果消息发布到哪个话题 (默认/vision/detections)

- target_topic  
指定这个节点把当前选中的主目标发布到哪个话题 默认/vision/target_point  

- min_area   
指定检测轮廓的最小面积阈值 过滤图像里的很多小碎块、小噪声 /一个颜色区域至少要多大，才有资格被认为是球

- orange_label  
指定橙色球在检测结果里使用的类别名称  
当节点识别出橙球时，会在检测结果或调试信息里给这个目标打一个标签 这个标签就是这个参数  

- blue_label  
指定蓝色球在检测结果里使用的类别名称
-------------------

```python
image_topic = self.get_parameter('image_topic').value  
debug_image_topic = self.get_parameter('debug_image_topic').value  
detection_topic = self.get_parameter('detection_topic').value  
target_topic = self.get_parameter('target_topic').value  
self.min_area = int(self.get_parameter('min_area').value)
self.orange_label = self.get_parameter('orange_label').value
self.blue_label = self.get_parameter('blue_label').value
```
### 读取参数
- image_topic  
把 image_topic 参数值取出来，保存到局部变量 image_topic  
- debug_image_topic  
取调试图话题名  

- detection_topic  
取检测结果话题名  

- target_topic = ...
取目标点话题名

- min_area = int(...)  
取 min_area 参数，并转成整数，保存成成员变量  
**注意**： 为什么写成 self.min_area  
           因为后面别的函数也要用（extract_ball_candidates）  

- self.orange_label  
保存橙球标签名

- self.blue_label  
保存蓝球标签名
-------------
```python
self.bridge = CvBridge()
```
创建一个图像转换工具对象 
ROS 图像消息 → OpenCV 图像
OpenCV 图像 → ROS 图像消息式

```python
self.image_sub = self.create_subscription(
Image, image_topic, self.image_callback, 10
        )
self.detection_pub = self.create_publisher(
Detection2DArray, detection_topic, 10
        )
self.target_pub = self.create_publisher(
PointStamped, target_topic, 10
        )
self.debug_pub = self.create_publisher(
Image, debug_image_topic, 10
        )
```
- self.image_sub    
订阅类型为 Image 的话题  
话题名是 image_topic  
每收到一帧图像，就调用 self.image_callback  
队列长度 10    
是这个节点的输入口  

- detection_pub  
创建一个发布器，用来发：  
类型：Detection2DArray  
话题：detection_topic  

- target_pub  
创建目标点发布器  
后面会发布当前主目标  
- debug_pub  
创建调试图像发布器  
后面会发布画好框的图  
------------
```python
self.get_logger().info(f'Perception node started. Subscribe image: {image_topic}')
```
打印启动日志 验证节点启动成功

-----------------------------
extract_ball_candidates 提取橙球函数
=
```python
def extract_ball_candidates(
        self,
        mask: np.ndarray,
        frame: np.ndarray,
        class_id: int,
        class_name: str,
        color: Tuple[int, int, int],
    ) -> List[Tuple[Detection2D, dict]]:
```
传入参数
- 一张颜色掩膜
- 原图
- 类别 id (类别编号：1 表示橙球 2 表示蓝球)
- 类别名
- 画框颜色  

**输出：一个列表 列表里每项是 (Detection2D, 附加信息字典)**  
返回所有候选球  
```python
contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        results = []
```
找轮廓 在掩膜图里找白色连通区域的轮廓  
*RETR_EXTERNAL 表示只取最外层轮廓*  
准备一个空列表，用来保存结果

-----------------
```python
for cnt in contours:
```
遍历每个轮廓

--------------
```python
area = cv2.contourArea(cnt)
if area < self.min_area:
   continue
```
计算轮廓面积 如果太小，直接跳过

---------------
```python    
perimeter = cv2.arcLength(cnt, True)
if perimeter < 1e-5:
   continue
```
算轮廓周长

-----------
```python
circularity = 4.0 * math.pi * area / (perimeter * perimeter)
            if circularity < 0.45:
                continue
```
计算圆形度   
球在图像里大致是圆形，如果圆形度太小，就认为它不像球
圆形度计算公式：
$$
4πA / P²
$$

-----------------------------
```python
x, y, w, h = cv2.boundingRect(cnt)
cx = x + w / 2.0
cy = y + h / 2.0
```
求外接矩形和中心
左上角坐标 (x, y)  
宽高 (w, h)  
中心点 (cx, cy)  

---------------------
```python
det = Detection2D()
det.header = Header()
```
构造单个检测框消息  
新建一个单目标检测消息

---------------------

```python
det.bbox.center.position.x = float(cx)
det.bbox.center.position.y = float(cy)
det.bbox.size_x = float(w)
det.bbox.size_y = float(h)
```
把框中心和框大小写进去  

-----------------
```python
hypo = ObjectHypothesisWithPose()
hypo.hypothesis.class_id = str(class_id)
hypo.hypothesis.score = float(min(1.0, 0.5 + circularity / 2.0))
det.results.append(hypo)
```
构造类别假设  
**class_id**
- 1代表橙球
- 2代表篮球  

**score**
简单用圆形度构造一个分数
越像圆，分数越高  
最高不超过 1.0  
然后把这个类别结果加到检测消息里  

---------------
```python
results.append((
   det,
   {
      'class_name': class_name,
      'cx': cx,
      'cy': cy,
      'w': w,
      'h': h,
      'area': area,
      'circularity': circularity,
   }
))
```
保存结果  
保存两部分信息：
- 第一部分 det  
正式要发布的 Detection2D  
- 第二部分字典  
额外内部信息，用来：后面选目标 调试画图 取面积排序  

------------------
```python
cv2.rectangle(frame, (x, y), (x + w, y + h), color, 2)
```
在原图上画框 在图像上画包围框
```python
cv2.putText(
      frame,
      f'{class_name} a={int(area)}',
      (x, max(20, y - 5)),
      cv2.FONT_HERSHEY_SIMPLEX,
      0.5,
      color,
      2,
      )
```
在框旁边写文字，比如：  
orange_ball a=532  
表示：  
这个球类别是什么 面积是多少

-------------------------

image_callback 图像回调函数
=
```python
def image_callback(self, msg: Image) -> None:
```
每收到一帧图像，这个函数就会自动执行  
参数 msg 就是收到的 ROS 图像消息
```python
try:
   frame = self.bridge.imgmsg_to_cv2(msg, desired_encoding='bgr8')
except Exception as e:
   self.get_logger().error(f'cv_bridge convert failed: {e}')
      return
```
**ROS 图像转 OpenCV 图像**  
指定转换结果是 OpenCV 常用的 BGR 8 位三通道格式  
(desired_encoding='bgr8')
```python
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
```
转 HSV 颜色空间

----------------------
```python
orange_lower = np.array([5, 120, 120])
orange_upper = np.array([25, 255, 255])

blue_lower = np.array([95, 100, 80])
blue_upper = np.array([130, 255, 255])
```
定义颜色范围

------------------
```python
orange_mask = cv2.inRange(hsv, orange_lower, orange_upper)
blue_mask = cv2.inRange(hsv, blue_lower, blue_upper)
```
按颜色生成二值  
落在范围内的像素设成白色 不在范围内的设成黑色  
**orange_mask：哪里像橙色 blue_mask：哪里像蓝色**

-------------------
```python
orange_mask = self.clean_mask(orange_mask)
blue_mask = self.clean_mask(blue_mask)
```
清理掩膜噪声
self.clean_mask详见前面

------------------
```python
orange_dets = self.extract_ball_candidates(
   mask=orange_mask,
   frame=frame,
   class_id=1,
   class_name=self.orange_label,
c  olor=(0, 140, 255),
)
 
blue_dets = self.extract_ball_candidates(
   mask=blue_mask,
   frame=frame,
   class_id=2,
   class_name=self.blue_label,
   color=(255, 0, 0),
)
```
处理橙色球和蓝色球的二值图  

- mask=orange_mask：要处理的掩膜

- frame=frame：原图，用来画框

- class_id=1：类别编号(1:橙色球 2:蓝色球)

- class_name=self.orange_label：类别名称

- color=(0, 140, 255)：画框颜色，橙色风格

--------------------------
```python
all_dets = orange_dets + blue_dets
```
合并所有检测结果 把橙球检测列表和蓝球检测列表拼起来

---------------------
```python
det_array = Detection2DArray()
```
构造检测结果消息  
创建一个“多目标检测结果”消息对象

----------------------
```python
det_array.header = msg.header
```
把原图像消息的头部复制过来，保持时间戳等信息一致

--------------------
```python
det_array.detections = [d[0] for d in all_dets]
self.detection_pub.publish(det_array)
```
因为 extract_ball_candidates 返回的是：(Detection2D, metadata_dict)  
d[0] 就是检测框消息本体  
这一句是在取所有检测框  
publish(det_array)  
把全部检测结果发到 /vision/detections  

-----------------------------------
```python
orange_candidates = [d for d in orange_dets]
if orange_candidates:
```
选择当前主目标  
先取所有橙球候选  
如果至少检测到一个橙球，就继续选目标  

------------------
```python
orange_candidates.sort(key=lambda x: x[1]['area'], reverse=True)
best = orange_candidates[0][1]
```
按面积从大到小排序  
取面积最大的橙球作为当前主目标

------------------
```python
target_msg = PointStamped()
target_msg.header = msg.header
``` 
构造目标点消息  
创建目标点消息，并继承原图像的头信息

------------------
```python
img_h, img_w = frame.shape[:2]
cx = best['cx']
cy = best['cy']
area = best['area']
```
读取：  
图像宽高  
当前目标球中心点  
当前目标面积  

------------------
```python
norm_x = (cx - img_w / 2.0) / (img_w / 2.0)
norm_y = (cy - img_h / 2.0) / (img_h / 2.0)
```
把图像坐标归一化到 [-1, 1] 附近

- norm_x  
表示目标在图像中相对中心的水平偏差：  
左边是负  
右边是正  
正中间接近 0

- norm_y
表示垂直偏差：  
上面是负  
下面是正  

------------------
```python
target_msg.point.x = float(norm_x)
target_msg.point.y = float(norm_y)
target_msg.point.z = float(area)
self.target_pub.publish(target_msg)
```
这里用 PointStamped 存了一个“简化目标信息”：  
x：水平偏差  
y：垂直偏差  
z：面积  
**注意：这里不是严格物理三维坐标，而是为了让控制节点好用**  
执行节点可以根据：  
x 决定向左转还是向右转  
z 判断目标是不是已经很近了  

------------------
```python
cv2.circle(frame, (int(cx), int(cy)), 8, (0, 255, 255), -1)
```
在调试图上画目标点  
在当前目标球中心画一个黄色小圆点


------------------
```python
cv2.putText(
    frame,
    'TARGET',
    (int(cx) + 10, int(cy) - 10),
    cv2.FONT_HERSHEY_SIMPLEX,
    0.7,
    (0, 255, 255),
    2,
)
```
在目标旁边写上 TARGET，方便看调试图时知道它选中了哪个球

------------------
```python
try:
    debug_msg = self.bridge.cv2_to_imgmsg(frame, encoding='bgr8')
    debug_msg.header = msg.header
    self.debug_pub.publish(debug_msg)
except Exception as e:
    self.get_logger().error(f'debug image publish failed: {e}')
```
发布调试图  
把 OpenCV 图像转回 ROS 图像消息  
publish(debug_msg)  
发布到 /vision/debug_image  
这样你可以用 rqt_image_view 看带框结果  










 
