### Confusion Matrix

| Confusion Matrix |Predicted Positive|Predicted Negative|
| ----------------:| ----------------:| ----------------:|
| Actual Positive  | TP               | FN               |
| Actual Negtive   | FP               | TN               |

**1.Basic evaluation:**

TPR: ture positive rate:  TP/(TP+FN) (equal to recall)

FPR: false positive rate: FP/(FP+TN)

TNR: true negative rate:  TN/(TN+FP)

FNR: false negative rate: FN/(FN+TP)

**2.Other evaluation:**

Accuracy=(TP+TN)/(TP+FN+FP+TN)

Missing rate=1-TPR=FNR

>漏警率(Missing Alarm): 漏警率(Missing Alarm)=1-Recall=FN/(TP+FN) 等同于Missing rate

Pecision: P=TP/(TP+FP) 

Recall  : R=TP/(TP+FN)

虚警率(False Alarm)  : 虚警率(False Alarm)=1-pecision=FP/(TP+FP)



### Pedestrain detection evaluation :FPPI and FPPW (benchmark)

>Dollár, Piotr, et al. "Pedestrian detection: A benchmark." Computer Vision and Pattern Recognition, 2009. CVPR 2009. IEEE Conference on. IEEE, 2009

FPPW： Given a set of N **negative samples image**, FPPW=FP/N (equal to false positive rate), that specifies the number of incorrect detections (FP), from the set of thousands or millions of negative examples in the testing set (notice that FPPW is not a rate but an absolute value). **a image is a sample**

It could be called per-window test, the axes of this plot are miss rate (MR) and false positives per window (FPPW). The FPPW working range depends on the dataset, but it is usually between 10−1 to 10−6. Notice that the axes are usually plotted in a log–log scale in order to focus on smallMR and FPPW values.

FPPI：Given a set of N image, **which exist or not exist detection objections**, FPPI=FP/N. (overlap when predicted bounding box hit actual bounding box is key.)

In this case, label is TP if it overlaps more than 50% (overlap we could determine) an annotated pedestrian in the groundtruth. Otherwise not overlap would be labeled as FN. The range tends to be between 10^(-2) to 10^(-0).

**FPPI is better than FPPW.** 

The typical curves for test are **receiver operating characteristic(ROC)**, detection-error-tradeoff(DET) and precision-recall


PR curve and missing rate VS FPPI would be as evaluation tools for perdestrain detection. There is a little different between them.

Almost pedestrain detection would use missing rate VS FPPI (ROC curve) as a evaluation.

#Using missing rate VS FPPI

## UDN and ContDeepNet (2014)
### 1、Unified deep net(UDN)

Dataset: Caltech and ETH

Evaluation tool: missing rate VS FPPI

**Best performace: average missing rate 39% in the Caltech-Test dataset.**

> Ouyang, Wanli, and Xiaogang Wang. "Joint deep learning for pedestrian detection." Proceedings of the IEEE International Conference on Computer Vision. 2013.

### 2、ContDeepNet

Dataset: Caltech, ETHZ and TUD-Brussels

Evaluation tool: missing rate VS FPPI

**Best performace: average missing rate 45% in the Caltech-Test dataset.**

>Zeng, Xingyu, Wanli Ouyang, and Xiaogang Wang. "Multi-stage contextual deep learning for pedestrian detection." Proceedings of the IEEE International Conference on Computer Vision. 2013.

#Using PR curve

## DPM (2014)


### Objection detection

Dataset: INRIA

Evaluation tool: PR-curve

**Best performace: average percision rate 37.6 for person class in INRIA**

>Felzenszwalb, Pedro F., Ross B. Girshick, and David McAllester. "Cascade object detection with deformable part models." Computer vision and pattern recognition (CVPR), 2010 IEEE conference on. IEEE, 2010.


### About Occluded people

>Tang, Siyu, Mykhaylo Andriluka, and Bernt Schiele. "Detection and tracking of occluded people." International Journal of Computer Vision 110.1 (2014): 58-69.




#Survey about pedestrain protection system(PPS)

>Gerónimo, David, and Antonio M. López. Vision-based pedestrian protection systems for intelligent vehicles. Springer, 2014.

##Candidates Generation

###2D-Based Approaches

Sliding windows in different scale(Could be input image or detection window, but scaling input image would be better than detection window, because the local image features in a resizing detection windowmust be also resizeable, while in the image-pyramid scanning they are not restricted.) depend on pedestrain modle size. 

**Problem:**
This would generate hundreds of thousands to millions of windows, depend on the sampling step and the minimun window size.

**Solution:** 

1.Reduce the top 1/3 of the image (could avoid perspective-incorrect candidates such as small-sized windows in near road positions.);

2.**flat world assumption** defines the techniques that restrict the candidate search to the ground plane without any other information than the initial camera position with respect to the plane.(could avoid urban scenes)

Somebody considers correcting the vertical image position by relying on the detection of horizontal edges oscillations depend on the previous frams to solve 
this problem in urban scenes.

3.Using **implicit shape model(ISM)** classification technique, matching the ones in a pedestrian model vote to promising location in the image likely to contain pedestrians.

4.coarse-to-fine search 

###3D-Based Approaches
...
###Motion-Based Approaches
Inter-frame motion and optical flow

##Classification
###Pedestrian Models
Holistic considers pedestrains as a whole.

parts-based methods  pedestrians consist of a set of body-inspired part, anatomy-based pr location-based, complemented with full body(holistic) as well.

parts-based methods could explicity incorporate multi-view, multi-resolution, and occlusion modeling.

###Pedestrian Classifiers

...
###Holistic Models:Focus on the Features

A hierachy would be used in  coarse-to-fine search to captures pedestrian silhouette.

**chamfer distance transform**
>Borgefors, Gunilla. "Distance transformations in digital images." Computer vision, graphics, and image processing 34.3 (1986): 344-371.

**the implicit shape model (ISM)** Original Image->Keypoint detection(SIFT, Harris-Laplace and shape context)->Matched codebook->Probabilistic voting->Maxima Selection->Back projection of masks

>Leibe, Bastian, Ales Leonardis, and Bernt Schiele. "Combined object categorization and segmentation with an implicit shape model." Workshop on statistical learning in computer vision, ECCV. Vol. 2. No. 5. 2004.

>Leibe, Bastian, and Bernt Schiele. "Interleaving object categorization and segmentation." Cognitive Vision Systems. Springer Berlin Heidelberg, 2006. 145-161.

Some Feature :Haar-like, LBP, HOG and so on.
