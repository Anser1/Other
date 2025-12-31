# IOU及演变

## IOU
<img width="790" height="536" alt="image" src="https://github.com/user-attachments/assets/b131566b-7947-4a72-9070-96d6656da945" />

## 代码实现
    import numpy as np
    def IoU(box1, box2):
        # 计算中间矩形的宽高
        in_w = min(box1[2], box2[2]) - max(box1[0], box2[0])
        in_h = min(box1[3], box2[3]) - max(box1[1], box2[1])
    
        # 计算交集、并集面积
        inter = 0 if in_w <= 0 or in_h <= 0 else in_h * in_w
        union = (box2[2] - box2[0]) * (box2[3] - box2[1]) +\
                (box1[2] - box1[0]) * (box1[3] - box1[1]) - inter
        # 计算IoU
        iou = inter / union
        return iou
    
    if __name__ == "__main__":
        box1 = [0, 0, 6, 8]  # [左上角x坐标，左上角y坐标，右下角x坐标，右下角y坐标]
        box2 = [3, 2, 9, 10]
        print(IoU(box1, box2))
优点：
- 具有尺度不变性；
- 满足非负性；
- 满足对称性；
缺点：
- 预测框与真实框之间不相交的时候，如果|A∩B|=0，IOU=0，无法进行梯度计算；
- 相同的IOU反映不出实际预测框与真实框之间的情况，虽然这三个框的IoU值相等，但是预测框与真实框之间的相对位置却完全不一样；
  <img width="856" height="266" alt="image" src="https://github.com/user-attachments/assets/cf3daf1b-6222-4e54-b8b7-0c9d94f1b506" />

也就是说，IoU 初步满足了计算两个图像的几何图形相似度的要求，简单实现了图像重叠度的计算，但无法体现两个图形之间的距离以及图形长宽比的相似性。

## GIOU
GIoU(Generalized Intersection over Union) 相较于 IoU 多了一个“Generalized”，通过引入预测框和真实框的最小外接矩形来获取预测框、真实框在闭包区域中的比重，从而解决了两个目标没有交集时梯度为零的问题。
引入了最小封闭形状C (可以把A，B包含在内)
<img width="294" height="338" alt="image" src="https://github.com/user-attachments/assets/08530c50-27ee-4fb5-a726-6c50aa2a2fe6" />

公式定义如下：

<img width="384" height="84" alt="image" src="https://github.com/user-attachments/assets/d540e6dd-0d0f-4b83-9a80-48b217fa9d3a" />

其中C是两个框的最小外接矩形的面积，原有 IoU 取值区间为[0,1]，而 GIoU 的取值区间为[-1,1] ；在两个图像完全重叠时IoU=GIoU=1，当两个图像不相交的时候IoU=0，GIOU=-1；

## 代码实现
    import numpy as np
    def GIoU(box1, box2):
        # 计算两个图像的最小外接矩形的面积
        x1, y1, x2, y2 = box1
        x3, y3, x4, y4 = box2
        area_c = (max(x2, x4) - min(x1, x3)) * (max(y4, y2) - min(y3, y1))
    
        # 计算中间矩形的宽高
        in_w = min(box1[2], box2[2]) - max(box1[0], box2[0])
        in_h = min(box1[3], box2[3]) - max(box1[1], box2[1])
    
        # 计算交集、并集面积
        inter = 0 if in_w <= 0 or in_h <= 0 else in_h * in_w
        union = (box2[2] - box2[0]) * (box2[3] - box2[1]) + \
                (box1[2] - box1[0]) * (box1[3] - box1[1]) - inter
        # 计算IoU
        iou = inter / union
    
        # 计算空白面积
        blank_area = area_c - union
        # 计算空白部分占比
        blank_count = blank_area / area_c
        giou = iou - blank_count
        return giou
    
    if __name__ == "__main__":
        box1 = [0, 0, 6, 8]
        box2 = [3, 2, 9, 10]
        print(GIoU(box1, box2))

优点：
- 与IoU只关注重叠区域不同，GIOU不仅关注重叠区域，还关注其他的非重合区域，能更好的反映两者的重合度；
- GIOU是一种IoU的下界，取值范围[ − 1 , 1 ] 。在两者重合的时候取最大值1，在两者无交集且无限远的时候取最小值-1。因此，与IoU相比，GIoU是一个比较好的距离度量指标，解决了不重叠情况下，也就是IOU=0的情况，也能让训练继续进行下去。
缺点：
- 但是目标框与预测框重叠的情况依旧无法判断：
<img width="531" height="263" alt="image" src="https://github.com/user-attachments/assets/41fd3288-5a3b-4164-bacc-e98df105cabd" />

## DIOU


------待补充





## IOU Loss的提出
通过下面例子可以看出：
<img width="861" height="382" alt="image" src="https://github.com/user-attachments/assets/6b2cbb7d-e850-41ad-b662-e4c768313ab9" />

在上诉图像中，可以看出图中所有目标的L2 Loss都一样，但是第二个的IoU显然是要大于第一个，第三个的IoU显然是要大于第二个，由图像分析可知，第三个矩形框的检测结果相对来说是最好的。
​ 当所有目标的L2 Loss是一样，但IoU却存在差异。通过4个点回归坐标框的方式是假设4个坐标点是相互独立的，没有考虑其相关性，实际4个坐标点具有一定的相关性。因此引入了IoU Loss。

## iou Loss
