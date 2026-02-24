---
title: "Object detection on different leaf types"
date: 2026-02-10T10:00:00+01:00
---

<a href="https://github.com/AkkuRam/plant-detection">
  <i class="fab fa-fw fa-github"></i> Plant Detection 
</a>

## Dataset

This dataset was obtained from Kaggle, contained over 50k images. Moreover, for the
model training this dataset was split into training (~36k) and test images (~17k).

[Kaggle Dataset](https://www.kaggle.com/datasets/sebastianpalaciob/plantvillage-for-object-detection-yolo/data#)


## Task

The main task was to perform object detection to identify 38 different leaf types. The ground truth bounding boxes, which were the YOLO annotation were given in the format (centerx, centery, width, height).

In terms of preprocessing, the images were resized to 256x256 pixels, where Gaussian Blur was applied 
to preprocess the salt/pepper noise present in the image. Moreover, the images were normalized as a final step.

## Regressor Model

Here the base model used was resnet50, where this is the model for the bounding box predictions.
Where the final layer has 4 outputs representing the format (centerx, centery, width, height) representing the 
bounding box.

```python
self.regressor = 
nn.Sequential(
    nn.Linear(base_model.fc.in_features, 256),
    nn.ReLU(),
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.Linear(128, 64),
    nn.ReLU(),
    nn.Linear(64, 32),
    nn.ReLU(),  
    nn.Linear(32, 4),
    nn.Sigmoid()
)
```

This format (centerx, centery, width, height) is later converted into the 4 corners as the bounding box
to be displayed on the test images. This is done through the following method:

```python
def yolo_to_xyxy(box, img_w, img_h):
    cx, cy, w, h = box

    cx *= img_w
    cy *= img_h
    w  *= img_w
    h  *= img_h

    x1 = int(cx - w / 2)
    y1 = int(cy - h / 2)
    x2 = int(cx + w / 2)
    y2 = int(cy + h / 2)

    return x1, y1, x2, y2
```

Using the above method, the YOLO format is converted to the 4 respective corners to display the bounding boxes
as seen in the batch results section below.

## Batch results

- Ground truth (green)
- Model predictions (red)
- Here 500 images were used for training and test to save computational power
- The batch size was 8 images, where in total there were 63 total batches

![Leaf](/images/leaf_detection.png)

## Running the file

- To run the file execute this command in the terminal "python -m src.train_pipeline"