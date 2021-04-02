**Thanks to Kaggle and hosts for this very interesting competition with a dataset too challenging. This has been as always a great collaborative effort and please also give your upvotes to @fatihozturk @socom20 @avsanjay . Congrats to Winners**


### TLDR
As @fatihozturk mentioned in this [post](https://www.kaggle.com/c/vinbigdata-chest-xray-abnormalities-detection/discussion/229724), Here is how we prepared our final submission 0.354/0.314 PublicLB/PrivateLB also had 0.330/0.321 PublicLB/PrivateLB (which we didnt selected) . The main magic for this competition was Ensembling and also there were lots of other tricks. We didnt had clear CV strategy, this was our main problem.

**CV strategy** :  This was our main challenge. We didn't have an unique validation scheme. Since we started the competition as different teams, we all were using different CV splits to train our models and to build our separated best ensembles. Finally we could overcome this difficulty by using the public LB as an extra validation dataset. We felt that this strategy was a bit risky, so we also retrained some of our best models using a common validation split and then we ensemble them. This last approach didn't have the best score on the private LB. We think that our last ensemble using the public LB as a validation source helped out final solution to fit better to the consensus method used to build the test dataset.

### Our Strategy
Our strategy for the final submission was 3 stages :
* Fully Validated Stage
* Partially Validated Stage
* Invalidated Stage (Fully LB based)
It was designed Since we didn't had a clear Validation scheme

### Models we used for Ensembling
- **Fully Validated Stage:**
  - Detectron2 Resnet50 (notebook by @corochann, Thanks)
  - YoloV5
  - EffDetD2
  - @nxhong93 (https://www.kaggle.com/nxhong93/yolov5-chest-512)

- **Partially Validated:**
  - YoloV5 5Folds
  - ClassWise model (i didnt worked but input as a low weighted)


- **Invalidated Stage:**
  - 2 EffdetD2 model  
  - 4 YOLOV5 (2x w/ TTA , 2x w/o TTA) by @awsaf49
  - 16 Classes Yolo model
  - New Anchor Yolo
  - Detectron2 Resnet50 (notebook by @corochann, Thanks)

### Ensembling Strategy


![enter image description here](https://i.postimg.cc/CMjP3z8J/Untitled-Document-1.png)


We used @zfturbo ’s ensembling repo. https://github.com/ZFTurbo/Weighted-Boxes-Fusion) and used above approach. 

In the Invalidated Stage, finally we use WBF+psum. p_sum was one of tricks in Modified WBF.
@socom20 has a modified WBF ensembling method, p_sum improved our Public from 0.319 to 0.331. But the Private seems to stand the same (i.e. 0.301) .

**Modified WBF:**  It also has more flexibility than WBF and also have 4 methods, we found only 2 of them were useful. 
- "p_det_weight" or as an alias "p_det_weight_pmean", has a behavior similar to WBF, it finds the set of bboxes with iou > thr, and then weights them using their detection probabilities (detection confidences). The new p_det is an average of the detection probabilities of detections with iou > thr.
- "p_det_weight_psum": does the same with the bboxes. In this case, the new p_det is a sum of the detection probability of bboxes with iou > thr. 
 Because of the sum of the p_dets, "p_det_weight_psum" needs a normalization step after a clean_predictions.
 
In our final submission Invalidated subs were weighted much more , which overfitted at PublicLB (i.e 0.354/0.314 Public/Private), if we didnt ( 0.330/0.321 Public / Private ) 

### Some Findings
- We used this [kernel](https://www.kaggle.com/shonenkov/bayesian-optimization-wbf-efficientdet) to find our best iou and skip_box_thr for ensembling.
we found iou somewhere between 0.33 to 0.4 is fine and we kept skip_box_thr as 0.0 or 0.001

![enter image description here](https://i.ibb.co/BgsPSBC/Vin-Big-Data-CV-Bayesian-Kaggle-3-31-2021-6-13-05-PM.png)
![enter image description here](https://i.ibb.co/Bg9WVrn/Vin-Big-Data-CV-Bayesian-Kaggle-3-31-2021-6-12-56-PM.png)

- After spending more time on the calculation of the competition metric. We also realized “No Penalty For Adding More Bbox” as @cdeotte explains clearly here: https://www.kaggle.com/c/vinbigdata-chest-xray-abnormalities-detection/discussion/229637 This was key to big jumps I guess. Having more confident boxes is good but at the same time having many low confidence boxes can only improve this metric…. It did worked for our single network. But for our ensemble this didn't.

![enter image description here](https://i.postimg.cc/BnL1z77F/photo-2021-03-31-18-37-48.jpg)

- For the last ten days, we spent analyzing our good and bad submissions both by submitting and also visually inspecting predicted boxes. I found out that our improved subs were indeed improving for almost all classes including common and rare ones.

![enter image description here](https://i.ibb.co/k0NRCXJ/photo-2021-04-01-00-30-35.jpg)

### Things didn't worked 
- Multi Label Classifier Filtering: We trained a resnet200d network with output dim as 14 (Since we have 14 classes). We used sigmoid as activation for the last dim because we need the probability of each disease happening for that specific image. But didnt helped. Since we were filterring by removing all boxes of a class which is less than a threshold. We also tried one of the Chexpert solution Attention pooling(PCAM) to do it. But didnt worked. 
- Bbox Filter: We trained a EfficientNetB6. Which classifies whether the detected boxes are belonging to same classes. The model couldnt understand the different between each disease (AUC:0.71)
- Usage of External Dataset: We couldnt use External Dataset (we only tried to use NIH) in much efficient way. We first train the backbone as a Multi-Label Classifier. and transferred the weights of the backbone to the Backbone of Detection model.
- ClassWise Model: Over here we trained different models for each classes. In this case some classes improved but decreased. But overall was an improvement. It would have work better if we have proper CV strategy. But we didnt, So we included it in Partially Validated Stage.

### Models we choose.
- Best at local validation (around 0.45+ mAP locally), Public: 0.300, private: 0.287
- Best at LB. Public: 0.354, private: 0.314
- Our best submission in private was 0.321 which was 0.330 in Public
