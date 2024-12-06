# COSMOs: Classification, Object detection, Segmentation MOduleS

## **Introduction**
This repo provides tools for common computer vision tasks.


| Tasks          | Subtasks | Defined Format | Visualization | Format Conversion | Output Analysis |
| -              | -        | -      | -             | -                 | -               |
| Classification | binary<sup>1</sup><br> binary-background<sup>2</sup><br> multi-class<sup>1</sup><br> multiclass-background<sup>2</sup><br> multi-binary<sup>3</sup><br> | [single_label](./example/classification/data/single_label.json)<sup>1</sup><br> [single_label_background](./example/classification/data/single_label_background.json)<sup>2</sup><br> [multi_label](./example/classification/data/multi_label.json)<sup>3</sup> | - | - | [metrics + plotting + export](example/classification/output_analysis.ipynb) |
| Detection      | - | [coco](./example/detection/data/coco)<br> [voc](./example/detection/data/voc)<br> [yolo](./example/detection/data/yolo)<br> [**GENERAL**](./example/detection/data/general.json)<br> | [ALL types](example/detection/s1_visualization_gt_and_pd.ipynb) | [between ANY two types](example/detection/s2_format_conversion.ipynb) | [metrics + plotting + export](example/detection/s3_output_analysis.ipynb) |
| Segmentation   | instance<sup>1</sup><br> semantic<sup>2</sup><br> | [coco](example/segmentation/data/coco)<sup>1+2</sup><br> [**GENERAL**](example/segmentation/data/general)<sup>1+2</sup> | [ALL types](example/segmentation/s1_visualization_gt_and_pd.ipynb) | [coco2general](example/segmentation/s2_format_conversion.ipynb) | [metrics + plotting + export](example/segmentation/s3_output_analysis.ipynb) |


+ Adding prediction results after the defined format can use the visualization and output analysis. All the formats with predictions are in `example/*/prediction`, e.g. [here](example/detection/prediction/general.json).


## **Motivation**

+ **[Classification]** Complicated tasks

| task                                          | label idx min | compute class-0 metrics | threshold optimization | data format |  
| -                                             | -             | -                       | -                      | -           |
| binary classification                         | 0             | V                       | V                      | [single_label](./example/classification/data/single_label.json)       |
| binary classification (cls-0 background)      | 1             |                         | V                      | [single_label_background](./example/classification/data/single_label_background.json) |
| multi-class classification                    | 0             | V                       |                        | single_label |
| multi-class classification (cls-0 background) | 1             |                         | V                      | single_label_background |
| multi-label classification (cls-0 background) | 0             |                         | V                      | [multi_label](./example/classification/data/multi_label.json)   |


+ **[Classification]** threshold optimization
    + `multi-class classification (cls-0 background)` checks whether prob-cls-0 < threshold, if yes, the pd-cls is pd[1:].argmax()
    + `multi-class classification (cls-0 background)` and `multi-label classification (cls-0 background)` take the mean of all optimized threshold for each foreground class


+ **[Object Detection]** Develop a [**GENERAL**](./example/detection/data/general.json) format to be the most convenient.

The formats can be summarized as following:
| format | extension | files     | type  | box                      | disadvantage |
| -      | -         | -         | -     | -                        | -            |
| coco   | .json     | 1         | int   | (xmin, ymin, w, h)       | get label of an image |
| yolo   | .txt      | len(imgs) | float | (cx, cy, w/2, h/2)       | visualization, compute metrics, etc. |
| voc    | .xml      | len(imgs) | int   | (xmin, ymin, xmax, ymax) | get class list |
| general| .json | 1 | int | (xmin, ymin, w, h) | **NO** |


+ **[Segmentation]** Develop a [**GENERAL**](./example/segmentation/data/general) format to be the most convenient.


| Includes         | Content | Advantage |
| -                | -       | -         |
| general.json     | Includes every imgs: path, contour, filled and boxes with class | Searching | 
| gt_contour_*.npy | (H, W) with {0, 1, ..., num_classes} int | Plotting |
| gt_filled_*.npy  | (num_classes, H, W) with 0 or 1 int values | Compute IOU for Metrics |
| *.jpg            | Raw data | - |


+ Segmentation prediction format: `(num_classes, H, W) with 0~1 float values (probability)`. e.g. [here](example/segmentation/prediction/instance/pd_filled_img1.npy)


## **Installation**
```bash
git clone https://github.com/bnbsking/COSMOs
pip install numpy matplotlib opencv-python scikit-learn
```

## **Quick Start - Classification**
+ Output Analysis:
    + Please conform your data format as either of one
        + [single_label](./example/classification/data/single_label.json)
        + [multilabel](./example/classification/data/multi_label.json)
        + [single_label_background](./example/classification/data/single_label_background.json)
    + The analysis pipeline is at [here](./cosmos/classification/output_analysis.yaml)
    + See more in the [example](./example/classification/output_analysis.ipynb)

```python
from cosmos.classification import ClassificationAnalysis

ClassificationAnalysis(
    ant_path = "example/classification/data/single_label.json",
    save_folder = "example/classification/output/single_label",
)
```


## **Quick Start - Object detection**
+ Format Conversion (see more in the [example](./example/detection/s2_format_conversion.ipynb))

```python
from cosmos.detection import coco2any

coco2any(
    tgt_foramt = "voc",
    img_folder = "example/detection/data/coco",
    ant_path = "example/detection/data/coco/coco.json",
    save_folder = "example/detection/output/visualization_gt_conversion/coco2voc"
)
```

or 

```python
from cosmos.detection import coco2general

coco2general(
    img_folder = "example/detection/data/coco",
    ant_path = "example/detection/data/coco/coco.json",
    save_path = "example/detection/output/visualization_gt_conversion/coco2general/general.json"
)
```

+ Visualization (see more in the [example](./example/detection/s1_visualization_gt_and_pd.ipynb))

```python
from cosmos.detection import show_coco

show_coco(
    img_name = "pic0.jpg",
    img_folder = "example/detection/data/coco",
    ant_path = "example/detection/data/coco/coco.json"
)
```

or

```python
from cosmos.detection import show_general

show_general(
    img_name = "pic0.jpg",
    ant_path = "example/detection/data/general.json",
)  # when the anntotation includes predictions it will be shown!
```

+ Output Analysis
    + Please use the above `Format conversion` to change data format as [general](./example/detection/data/general.json)
    + The analysis pipeline is at [here](./cosmos/detection/output_analysis.yaml)

```python
from cosmos.detection import DetectionAnalysis

DetectionAnalysis(
    ant_path = "example/detection/data/general.json",
    save_folder = "example/detection/output/metrics"
)
```


## **Quick Start - Segmentation**
+ Format Conversion (see more in the [example](example/segmentation/s2_format_conversion.ipynb))

```python
from cosmos.segmentation import coco2general

coco2general(
    img_folder = "example/segmentation/data/coco",
    ant_path = "example/segmentation/data/coco/coco.json",
    save_folder = f"example/segmentation/output/visualization_gt_conversion/coco2general"
)
```

+ Visualization (see more in the [example](example/segmentation/s1_visualization_gt_and_pd.ipynb))

```python
from cosmos.segmentation import show_coco

show_coco(
    img_name = "img1.jpg",
    img_folder = "example/segmentation/data/coco",
    ant_path = "example/segmentation/data/coco/coco.json"
)   # when the anntotation includes predictions it will be shown!
```

or

```python
from cosmos.segmentation import show_general

show_general(
    img_name = "img1.jpg",
    ant_path = "example/segmentation/data/general/general.json"
)
```

+ Output Analysis
    + Please use the above `Format conversion` to change data format as [general](./example/segmentation/data/general)
    + The analysis pipeline is at [here](./cosmos/segmentation/output_analysis_instance.yaml)

```python
from cosmos.segmentation import SegmentationAnalysis

SegmentationAnalysis(
    ant_path = "example/segmentation/prediction/instance/general.json",
    save_folder = "example/segmentation/output/metrics",
    task = "instance",
)
```

or

```Python
from cosmos.segmentation import SegmentationAnalysis

SegmentationAnalysis(
    ant_path = "example/segmentation/prediction/semantic/general.json",
    save_folder = "example/segmentation/output/metrics/semantic",
    task = "semantic"
)
```

## **Examples**
+ **[detection]**: format conversion workflow
![.](pictures/detection_workflow.png)

+ detection visualization
![.](pictures/detection_visualization.jpg)

+ confusion
![.](pictures/confusion.jpg)

+ prf curves
![.](pictures/prf_curves.jpg)