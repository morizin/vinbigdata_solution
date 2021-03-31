**Thanks to Kaggle and hosts for this very interesting competition with a biased dataset. This has been as always a great collaborative effort and please also give your upvotes to @fatihozturk @socom20 @avsanjay . Congrats to Winners**


### TLDR
As @Fatih mentioned in the [post](https://www.kaggle.com/c/vinbigdata-chest-xray-abnormalities-detection/discussion/229724), We prepared our 0.331/0.301 PublicLB/PrivateLB Score very easier. The main magic for this competition was Ensembling. The truth is we could achieve the score on LB >0.3 very easier by only ensembling

**CV strategy** :  This was our main problem. we didnt had much efficient and unique validation scheme. We all were using different CV split to attain diversity among the CV also. But the whole thing collapsed at the end to lower score in PrivateLB.

### Models we used for Ensembling
- 2 EffdetD2 model  @awsaf49
- 4 YOLOV5 (2x w/ TTA , 2x w/o TTA) by 
- 16 Class Yolo model
- New Anchor Yolo
- Detectron2 Resnet50 (notebook by @corochann, Thanks)

### Ensembling Strategy
![enter image description here](https://i.ibb.co/5WKvNyK/Simple-Ensemble-Lucidchart-3-31-2021-5-54-15-PM.png)

This Ensembling scheme gave a huge increase in both Public and Private LB. If we use only WBF we could only achieve 0.294 in Public LB. In these strategy, we got increase to 0.319/0.301 in PublicLB/PrivateLB .

**Ensemble Block** : *WBF + below ensemble method*

@socom20 has a modified WBF ensembling method, which improved our Public from 0.319 to 0.331. But the Private seems to stand the same (i.e. 0.301) .

### Some Findings
We used this [kernel](https://www.kaggle.com/shonenkov/bayesian-optimization-wbf-efficientdet) to find our best iou and skip_box_thr for ensembling.
we found iou somewhere between 0.33 to 0.4 is fine
![enter image description here](https://i.ibb.co/BgsPSBC/Vin-Big-Data-CV-Bayesian-Kaggle-3-31-2021-6-13-05-PM.png)
![enter image description here](https://i.ibb.co/Bg9WVrn/Vin-Big-Data-CV-Bayesian-Kaggle-3-31-2021-6-12-56-PM.png)

### Things didn't worked 
- Multi Label Classifier Filterring
- Bbox Filter

