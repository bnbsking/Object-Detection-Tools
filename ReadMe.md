Object detection format overview 

| - | VOC | YOLO | COCO |
| - | - | - | - |
| file | .xml per img | .txt per img | .json for all img \n D |
|      |              |              | D |
| format | xmin,ymin,xmax,ymax | cx,cy,w,h | xmin,ymin,w,h |
| type | int | float | int |
| classes | - | classes.txt | D['categories'] |
| raw shape | height,width | - | D['images'] |

+ VOC
