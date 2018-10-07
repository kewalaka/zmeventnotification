### Note

** I don't plan to provide support for these hooks. They are for reference only and require setup and some degree of know how **

This is just an example of how you can use the new `hook` feature of the notification server
to invoke a custom script on the event before it generates an alarm. This implements a hook script that detects
for persons in an image that raised an alarm before sending out a notification. 

There are two sample detection scripts. You can switch between them by changing the value of
`DETECTION_SCRIPT` in `detect_wrapper.sh`

**Both of these scripts require setup, please run from command line first and 
make sure deps are installed**

Please don't ask me questions on how to use them. Please read the comments and figure it out.

Try to keep the images less than 800px on the largest side. The larger the image, the longer
it will take to detect

### detect_hog.py: using OpenCV SVM HOG (very fast, not accurate)

You can manually invoke it to test:

```
./detect_hog.py --image image.jpg
```

The detection uses a very fast, but not very accurate OpenCV model (hog.detectMultiScale). 
The good part is that it is extremely fast can can be used for realtime needs. 
Fiddle with the settings in detect.py (stride/scale) to get more accuracy at the cost of speed.

### detect_yolo.py:  using OpenCV DNN with YoloV3 (much slower, accurate)

The detection uses OpenCV's DNN module and YoloV3 to predict multiple labels with score.

You can manually invoke it to test:

```
./detect_yolo.py --image image.jpg --config path/to/config/file --weight path/to/weights/file --label path/to/label/file
```


If you are using YOLO models, download the data files from:
* weights: https://pjreddie.com/media/files/yolov3.weights
* config: https://raw.githubusercontent.com/arunponnusamy/object-detection-opencv/master/yolov3.cfg
* labels: https://raw.githubusercontent.com/arunponnusamy/object-detection-opencv/master/yolov3.txt


### performance comparison

DNNs perform very well on a GPU. My ZM server doesn't have a GPU. 
On a Intel Xeon 3.16GHz 4Core machine, `detect.py` takes 0.2s while `detect_yolo.py` takes 2.4s. 


As always, if you are trying to figure out how this works, do this in 3 steps:

**STEP 1: Make sure the scripts(s) work**
- Run the python script manually with `--image <path/to/image/image.jpg>` to see if it works
- `./detect_wrapper.sh <eid>` --> make sure it downloads a proper image for that eid. Make sure it correctly invokes detect.py If not, fix it.
- Make sure the `IMAGE_PATH` you've chosen in `detect_wrapper.sh` is WRITABLE by www-data (or apache) before you move to step 2

**STEP 2: run zmeventnotification in MANUAL mode**
* `sudo zmdc.pl start zmeventnotification.pl`
*  change verbose to 1 in `zmeventnotification.ini`
*  `sudo -u www-data ./zmeventnotification.pl  --config ./zmeventnotification.ini`
*  Force an alarm, look at logs

**STEP 3: integrate with the actual daemon**

* You should know how to do this already