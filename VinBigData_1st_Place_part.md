
**Thanks to Kaggle and hosts for this very interesting competition with a too challenging dataset. This has been as always a great collaborative effort. Please give your upvotes to @fatihozturk @socom20 @avsanjay . Congrats to The Winners**


### TLDR
@fatihozturk explained in his [post](https://www.kaggle.com/c/vinbigdata-chest-xray-abnormalities-detection/discussion/229724) how was his path through this competition, this post is intented as an overall view of our final solution which achieved 0.354/0.314 PublicLB/PrivateLB. It is worth mentioning that we also had a better solution with 0.330/0.321 PublicLB/PrivateLB (which unfortunately we didn't select). As you all aready know the main magic we have used for this competition was our Ensembling procedure, also there were lots of other important findings that we mention in this post.

**CV strategy**:  This was our main challenge, we didn't have an unique validation scheme. Since we started the competition as different teams, we all were using different CV splits to train our models and to build our separated best ensembles. To overcome this difficulty we finally ended up by using the public LB as an extra validation dataset. We felt that this strategy was a bit risky, so we also retrained some of our best models using a common validation split and then we ensemble them (this last approach didn't have the best score on the private LB by itself). We think that our last ensemble, using the public LB as a validation source, helped out final solution to fit better to the consensus method used to build the test dataset. 

### Our Validation Strategy
Our strategy for the final submission can be divided in 3 stages :
* Fully Validated Stage
* Partially Validated Stage
* Not validated Stage (Just validated using the Public LB)

### Models we used for Ensembling
- **Fully Validated Stage:**
  - Detectron2 Resnet101 (notebook by @corochann, Thanks)
  - YoloV5
  - EffDetD2

- **Partially Validated:**
  - YoloV5 5Folds

- **Not Validated Stage:**
  - 2 EffdetD2 model  
  - 3 YoloV5 (1x w/ TTA , 2x w/o TTA)
  - 16 Classes Yolo model
  - 5 Folds YoloV5 by @awsaf49
  - New Anchor Yolo
  - Detectron2 Resnet50 (notebook by @corochann, Thanks)
  - YoloV5 @nxhong93 (https://www.kaggle.com/nxhong93/yolov5-chest-512)
  - Yolov5 with image size (640)

### Ensembling Strategy

To ensemble all the partial models we mainly used @zfturbo's ensembling repository https://github.com/ZFTurbo/Weighted-Boxes-Fusion. Our ensemble technic is shown in the following figure:

![enter image description here](https://i.postimg.cc/ncRwRrfg/2x-Page-1-5.png)

As you can see at the end of the Not Validated Stage, we use WBF+p_sum. This last blending method is a variant of WBF proposed by @socom20 and was intended to simulate the consensus of radiologists used in this particular test dataset. By using WBF+p_sum improved our Not Validated stage from 0.319 to 0.331 in the PublicLB.

**Modified WBF:**  It has more flexibility than WBF, it has 4 bbox blending technics, we found that only 2 of them were useful in this competition:
- "p_det_weight_pmean", has a behavior similar to WBF, it finds the set of bboxes with iou > thr, and then weights them using their detection probabilities (detection confidences). The new p_det (detection confidence) is an **average** over the detection probabilities of detections with iou > thr.
- "p_det_weight_psum": does the same with the bboxes. In this case, the new p_det is a **sum** over the detection probability of bboxes with iou > thr.
Because of the sum of the p_dets, "p_det_weight_psum" needs a final normalization step. The idea behind psum is to try to replicate "the consensus of radiologists" using the partial models. When adding the probabilities of detection, we weight more those boxes in which more models/radiologists agree.

In our final submission the not validated stage had a high weighing, that lead to an overfitted problem in our final ensemble we got  0.354/0.314 Public/Private. If we had reduced the weighting for this not validated stage, the LB score would have become 0.330/0.321 Public/Private as we previously have mentioned in the post.

### Some Interesting Findings
- We found that the public score was quite representative for the final PrivateLB. Because of this we thought our 3 stage Validation approach is good in our case. We found this by using our unique val set. Here's how we did it. We started a for loop of 500 iterations, In each iterations we sampled 300 images from our val set, and calculated mAP between a single model and ground truth (blue dots) at every 500 iterations. and we calculated mAP between a ensembled model and ground truth (orange dots) at every 500 iterations. For the graph below we understands that in a some of 300 images sample, in some cases ensemble model have low mAP than a single model. We found this was the problem happend when we have a better CV model but low LB and better LB and low CV. I think some might have encountered this problem.

![enter image description here](https://i.postimg.cc/0Q8V0WCG/photo-2021-04-08-16-09-42.jpg)

- After spending more time understanding the competition metric, we realized that there is No Penalty For Adding More Bbox as @cdeotte explains clearly here: https://www.kaggle.com/c/vinbigdata-chest-xray-abnormalities-detection/discussion/229637. Having more confident boxes is good but, at the same time, having many low confidence bboxes imploves the Recall and can improve the mAP as well. We tested this approach by using two models, a  Model A with a good mAP, and a Model B with a good Recall for all the classes. The folowing figure shows on the left the Precision vs Recall curves of Model A. It can be seen that many class tails end with a maximun recall = 0.7. We managed to add to Model A all the bboxes predicted for Model B which doesn't overlap with any bbox of predicted for Model A, the result is in the plot of the right. It can be seen that the main shape of the PvsR curves are still the same, but the tails now reach over recall=0.8. We didn't use this a[proach in our final model because the final ensemble aready had a very good recall for all the clases.

![enter image description here](https://i.postimg.cc/BnL1z77F/photo-2021-03-31-18-37-48.jpg)

- Last competition days, we spent analyzing the predictions made for the ensemble both by submitting and also visually inspecting predicted bboxes. We found that our improved submissions were indeed improving for almost all classes including the easy and the rare ones.

![enter image description here](https://i.ibb.co/k0NRCXJ/photo-2021-04-01-00-30-35.jpg)

### Things didn't worked 
- We used sigmoid as activation because we needed the probability of each disease and many images have more than one disease. The idea was to remove from the ensemble all the bboxes not predicted by this classifier model, this approach lead to reducing the performance.  We also tried a switch the pooling layer and to use PCAM (it is an attention pooling used in the [Top1 Solution of CheXpert](https://github.com/jfhealthcare/Chexpert)), but it didn't work either. 
- Bbox Filter: We trained a EfficientNetB6 which classifies whether the detected bboxes belong to selected classe or not just by seeing the crop selected by the ensemble. The model couldn't understand the different between each disease, perhaps it needed more information of the whole image.
- Usage of External Dataset: We tried to use NIH dataset. We first trained a backbone as a Multi-Label Classifier using NIH classes, and then we transfer its weights of the detector's backbone. Finally we trained only the detection heads using this competition's dataset. We couldn't detect any apreciable improvement in the particular detector score.
- ClassWise Model: we trained different models, one for each detection class. We used clsX + cls14 as each model's dataset. In this case each model had its own set of anchors prepared for detecting the class of interest. Some classes improved its CV performance but other didn't. Joining all the models we got a very good model, unfortunately the validation split used in this model was different form the others it was unreliable to add it to the Validated Stage so we decided to included it in Partially Validated Stage. But still didnt gave any improvements.

### Models we choose.
- Best at local validation (around 0.47+ mAP on CV), Public: 0.300, private: 0.287
- Best at LB. Public: 0.354, private: 0.314
- Our best submission in private was 0.321 which was 0.330 in Public

### Hardware we used
- 4x Quadro GV100
- 4x Geforce GTX 1080
- 4x Titan X
- Ryzen 9 3950x, 64GB of Ram Memory 
- Nvidia RTX 3080.
- Quadro RTX 6000
