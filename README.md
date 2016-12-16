# facedetection

## Required:
Pre-trained face detection model 
`https://www.dropbox.com/s/6isizo52zi671s4/shape_predictor_68_face_landmarks.dat.bz2?dl=0

Openface
`https://www.dropbox.com/sh/tr1itmet6g4j8gu/AACm6OV6g5VxhBcy7UqASPpQa?dl=0

dlib
`https://www.dropbox.com/sh/iee0vazuroga8du/AACK5B7kHG2_a5a83Zzqgeo2a?dl=0

##Step by step procedure:

Encode a picture using the HOG algorithm to create a simplified version of the image. Using this simplified image, find the part of the image that most looks like a generic HOG encoding of a face.

Figure out the pose of the face by finding the main landmarks in the face. Once we find those landmarks, use them to warp the image so that the eyes and mouth are centered.

Pass the centered face image through a neural network that knows how to measure features of the face. Save those 128 measurements.

Looking at all the faces we’ve measured in the past, see which person has the closest measurements to our face’s measurements. That’s our match!

--------------------------------------------------------------------
## Step 1

Make a folder called `./training-images/` inside the openface folder.

```
mkdir training-images
```

## Step 2 

Make a subfolder for each person you want to recognize. For example:

```
mkdir ./training-images/will-ferrell/
mkdir ./training-images/chad-smith/
mkdir ./training-images/jimmy-fallon/
```

## Step 3 

Copy all your images of each person into the correct sub-folders. Make sure only one face appears in each image. There's no need to crop the image around the face. OpenFace will do that automatically.

## Step 4

Run the openface scripts from inside the openface root directory:

First, do pose detection and alignment:

`./util/align-dlib.py ./training-images/ align outerEyesAndNose ./aligned-images/ --size 96`

This will create a new `./aligned-images/` subfolder with a cropped and aligned version of each of your test images.

Second, generate the representations from the aligned images:

`./batch-represent/main.lua -outDir ./generated-embeddings/ -data ./aligned-images/`

After you run this, the `./generated-embeddings/` sub-folder will contain a csv file with the embeddings for each image.

Third, train your face detection model:

`./demos/classifier.py train ./generated-embeddings/`

This will generate a new file called `./generated-embeddings/classifier.pkl`. This file has the SVM model you'll use to recognize
new faces.

At this point, you should have a working face recognizer!

## Step 5: Recognize faces!

Get a new picture with an unknown face. Pass it to the classifier script like this:

`./demos/classifier.py infer ./generated-embeddings/classifier.pkl your_test_image.jpg`

You should get a prediction that looks like this:

```
=== /test-images/will-ferrel-1.jpg ===
Predict will-ferrell with 0.73 confidence.
```

From here it's up to you to adapt the `./demos/classifier.py` python script to work however you want.

Important notes:
* If you get bad results, try adding a few more pictures of each person in Step 3 (especially picures in different poses).
* This script will _always_ make a prediction even if the face isn't one it knows. In a real application, you would look at the confidence score and throw away predictions with a low confidence since they are most likely wrong.

##Reference

`https://medium.com/@ageitgey/machine-learning-is-fun-part-4-modern-face-recognition-with-deep-learning-c3cffc121d78#.4tkz2bczp
