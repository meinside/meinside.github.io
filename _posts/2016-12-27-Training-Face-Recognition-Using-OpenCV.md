---
layout: post
title: Training Face Recognition using OpenCV
tags:
- opencv
- python
published: true
---

Here are what I did for training face recognition using OpenCV.

With these steps, I learned how to run `opencv_createsamples` and `opencv_traincascade`,

and also how hard it is to train the computer to recognize something.

----

# 1. The very beginning

I was learning OpenCV, and wanted to train something by my own hand.

There were hundreds of selfie photos of myself taken by [this way](https://blog.meinside.pe.kr/Take-a-Selfie-on-Every-Wakeup-from-Sleep-OSX/),

![my_faces](https://cloud.githubusercontent.com/assets/185988/21497844/f3d8689c-cc6a-11e6-8149-a0da02869170.png)

so I made up my mind to train my computer to recognize my face.

# 2. Refine training samples

Knowing not all my photos were usable for the training, I wanted to filter out unusable ones.

For the refinement, I ran following python script:

```python
#!/usr/bin/env python

import cv2

import glob
import os.path as path

PHOTOS_DIR = 'photos'
CHECKED_PHOTOS_DIR = 'checked_photos'

# mark faces on given image
def mark_face(cascade, filepath):
    image = cv2.imread(filepath, cv2.IMREAD_COLOR)
    faces = cascade.detectMultiScale(image, 1.3, 5)

    for (x, y, w, h) in faces:
        cv2.rectangle(image, (x, y), (x + w, y + h), (0, 0, 255), 2)

    return image

# XXX - Download: https://raw.githubusercontent.com/shantnu/Webcam-Face-Detect/master/haarcascade_frontalface_default.xml
default_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')

# iterate all the photos and mark rectangles on detected faces
filepaths = glob.glob(path.join(PHOTOS_DIR, '*.jpg'))
for filepath in filepaths:
    marked_image = mark_face(default_cascade, filepath)
    cv2.imwrite(path.join(CHECKED_PHOTOS_DIR, path.basename(filepath)), marked_image)
```

This script detects faces from photos in **PHOTOS_DIR** with pre-trained cascade file,
draws red rectangles on them, and saves the result photos in **CHECKED_PHOTOS_DIR**.

All the result photos with no rectangles, or with rectangles on wrong position would not be a good sample for the training,
so I deleted them from **PHOTOS_DIR**.

# 3. Generate positive/negative list files

I downloaded negative facial images from [here](https://www.bioid.com/download?path=BioID-FaceDatabase-V1.2.zip) and placed them in **NEGATIVE_PHOTOS_DIR**.
(I converted them into .jpg format!)

![negative_faces](https://cloud.githubusercontent.com/assets/185988/21497843/f3b56766-cc6a-11e6-8c97-18c1a8a9b9c7.png)

With the negative images, following script generates `positive.txt` and `negative.txt`:

```python
#!/usr/bin/env python

import cv2

import glob
import os
import os.path as path

PHOTOS_DIR = 'photos'
NEGATIVE_PHOTOS_DIR = 'negative_photos'

# list files for training
POSITIVE_LIST_FILENAME = 'positive.txt'
NEGATIVE_LIST_FILENAME = 'negative.txt'

# extract facial regions from given image,
# and return the largest
def detect_face(cascade, filepath):
    image = cv2.imread(filepath, 0)
    faces = cascade.detectMultiScale(image, 1.3, 5)

    face = None
    for (x, y, w, h) in faces:
        if face is None or w * h > face[2] * face[3]:
            face = (x, y, w, h)

    if face is None:
        return filepath, 0, None
    else:
        return filepath, 1, face

default_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')

# (positive) iterate all images
pfilepaths = glob.glob(path.join(PHOTOS_DIR, '*.jpg'))
print 'Iterating ' + str(len(pfilepaths)) + ' positive files...'
num_detected = 0
positive_file = open(POSITIVE_LIST_FILENAME, 'w')
for filepath in pfilepaths:
    fpath, count, region = detect_face(default_cascade, filepath)
    if region != None:
        num_detected += 1
        line = ' '.join((fpath, str(count), str(region[0]), str(region[1]), str(region[2]), str(region[3])))
        positive_file.write(line + '\n')
positive_file.close()

# (negative) iterate all images
nfilepaths = glob.glob(path.join(NEGATIVE_PHOTOS_DIR, '*.jpg'))
print 'Iterating ' + str(len(nfilepaths)) + ' negative files...'
negative_file = open(NEGATIVE_LIST_FILENAME, 'w')
for filepath in nfilepaths:
    negative_file.write(filepath + '\n')
negative_file.close()

# print result
print 'Total ' + str(num_detected) + '/' + str(len(pfilepaths)) + ' positive images, ' + str(len(nfilepaths)) + ' negative images'
```

`positive.txt` is filled up with lines which consist of positive image's filepath and facial positions.

`negative.txt` has negative images' filepaths in it.

# 4. Create samples

I ran following command with generated `positive.txt`:

```bash
$ opencv_createsamples -info positive.txt -vec training.vec -num 400
```

and got `training.vec` as a result.

# 5. Train

Finally, with generated `training.vec` and `negative.txt`, I ran:

```bash
$ mkdir -f result
$ opencv_traincascade -data result -vec training.vec -bg negative.txt -numPos 300 -numNeg 1500 -featureType HAAR -mode CORE -numStages 12 -maxFalseAlarmRate 0.5 -minHitRate 0.995
```

(Parameters may vary.)

Fortunately, there were no errors while running it.

I could find the final result: `cascade.xml` in **result** directory.

# 6. Verify the result

I got `cascade.xml`, and wanted to verify it if the training was successful.

I slightly modified the first python script:

```python
#!/usr/bin/env python

import cv2

import glob
import os
import os.path as path

PHOTOS_DIR = 'photos'
CHECKED_PHOTOS_DIR = 'checked_photos'

RESULT_DIRNAME = 'result'
TRAINED_CASCADE_XML_FILENAME = path.join(RESULT_DIRNAME, 'cascade.xml')

# mark faces on given image
def mark_face(cascade, filepath):
    image = cv2.imread(filepath, cv2.IMREAD_COLOR)
    faces = cascade.detectMultiScale(image, 1.3, 5)

    for (x, y, w, h) in faces:
        cv2.rectangle(image, (x, y), (x + w, y + h), (0, 0, 255), 2)

    return image

# delete previously marked files
for filepath in glob.glob(path.join(CHECKED_PHOTOS_DIR, '*.jpg')):
    os.remove(filepath)

my_cascade = cv2.CascadeClassifier(TRAINED_CASCADE_XML_FILENAME)

# iterate all images
filepaths = glob.glob(path.join(PHOTOS_DIR, '*.jpg'))
print 'Iterating ' + str(len(filepaths)) + ' files...'
num_detected = 0
for filepath in filepaths:
    image = mark_face(my_cascade, filepath)

    if image is None:
        print "No face in: " + filepath
    else:
        num_detected += 1
        print "Face detected in: " + filepath
        cv2.imwrite(path.join(CHECKED_PHOTOS_DIR, path.basename(filepath)), image)

# print result
print 'Total ' + str(num_detected) + '/' + str(len(filepaths)) + ' faces were marked in directory: ' + CHECKED_PHOTOS_DIR 
```

This script now draws rectangles on faces recognized by the newly-generated cascade file(`result/cascade.xml`).

Time to check the marked photos in **CHECKED_PHOTOS_DIR**!

# 8. The result is...

Some of the photos had red rectangles on correct positions,

![good1](https://cloud.githubusercontent.com/assets/185988/21496696/c54d5700-cc63-11e6-862f-c72d3ce889e8.jpg)
![good2](https://cloud.githubusercontent.com/assets/185988/21496697/c59b2a20-cc63-11e6-9b60-8259fba75488.jpg)

but others did not:

![bad1](https://cloud.githubusercontent.com/assets/185988/21496698/c5e0e1a0-cc63-11e6-97a7-a79010067dd9.jpg)
![bad2](https://cloud.githubusercontent.com/assets/185988/21496699/c6295cc8-cc63-11e6-8041-75de80407a1a.jpg)

The result was poorer than I expected.

Maybe the positive/negative photos were not perfect for this training, or the train parameters were not good enough.

# 9. Wrap-up

Now I can create samples and train OpenCV to recognize something, but the accuracy is not satisfying yet.

I have to learn more about this topic.

# 999. Reference

* [OpenCV Haar/cascade training 튜토리얼](http://darkpgmr.tistory.com/70)

