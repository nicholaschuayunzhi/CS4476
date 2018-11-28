<frontmatter>
  title: "Midterm Update"
  footer: footer.md
</frontmatter>

<include src="nav.md" boilerplate />

# Final Update 28/11/2018
We have applied our stitching algorithm to cave photos. Due to challenges faced with the photography and stitching, we have adjusted experiment 3 to do a cylindrical mapping instead. We also realised experiment 4 was crucial for the stitching of the panorama, and will be discussed first.

As a recap, our pipeline after the midterm update is:
```
Apply cylindrical warp on first image
stitch image = first image
for image in subsequent images
  1. Apply cylindrical warp on image
  2. Extract features from greyscale stitch image and image using SIFT
  3. Use KNN to match the features to identify corespondences
  4. Given correspondences, run RANSAC to find homography matrix
  5. Warp image onto reference frame of stitch image
  6. Do alpha blending between current image and image
  7. Set stitch image = merged image
```

# Photography
Photography was done by group member James Gormley.

A significant challenge for this project was the collection of data. There were several key issues, including a lack of photography skill on my part, and a lack of professional equipment. After three caving trips, the final one was able to provide mediocre quality picture sets which we were able to do some work with, issues that hampered the other trips include a lack of lighting, and poor images due to blurring.

Photography in caves can be especially problematic (something we are trying to help solve), but our issues were exacerbated by a lack of proper equipment. Taking 360 photos with a regular handheld camera is not the best approach as maintaining the camera’s position can be difficult.

Also, the nature of caves not having a flat surface from which to take the pictures further impacted the positioning of the camera. The required lighting equipment for caves is very expensive and difficult to transport through the cave, leading to photos with lighting conditions that were more inconsistent than necessary as lackluster equipment was used instead. In the end, pictures were obtained, but they were of lower quality than I would have liked.

# Experiment 4

### Pre-processing Images

<pic src="images/preprocess.jpg" width="410" alt="Logo">Poor Lighting Conditions</pic>
<pic src="images/cropped.png" width="300" alt="Logo">Cropped Image</pic>

As our single source of light is from the camera man's helmet, our images were mostly only illuminated at the center. Furthermore, we had little to none gradient information to work with for the dark areas. Thus the black ring was unimportant and could lead to artifacts between images when we try to blend them together.

We decided to crop out the black border of the image. As a slight adjustment for better croppng results, we checked the grayscale intensity and allowed our algorithm to crop intensities below 50. This gave us a tight box crop over the cave image.

### Image Adjustment
When attempting our stitching, we faced a few problems:
* incorrectly matched correspondences
* insufficient correspondences

<pic src="images/correspondences.png" width="900" alt="Logo">Stiched Image on Left, Incorrectly Matched Keypoints on Right</pic>

As can seen from above, correspondences were matched incorrectly. This is understandable due to the rockiness of the caves. It is difficult to distinguish rocks from one another.

Another problem we faced was that for many pictures there were insufficient number of matched correspondences to calculate a homograpy. As the light source is from the camera's origin, rock structures would form different shadows in different picture.

To overcome this, we did Contrast Limited Adaptive Histogram Equalization illumination (CLAHE) ilumination to improve the contrast in image to make rock structures more distinct and thus result in better correspondence matching.

<pic src="images/beforeclahe.png" width="300" alt="Logo">Original Image</pic>
<pic src="images/afterclahe.png" width="300" alt="Logo">After CLAHE Illumination</pic>

A thing to note is that we apply CLAHE illumination on the images for feature detection. However, the final image does not have the illumination applied to it as we find it to be visually unpleasing. This works as the image adjustment does not adjust the position of the features - just the gradient information of the image.

### Lowe's Ratio
Additionally, we increased the ratio used in our Lowe's Ratio test to allow for more keypoint matches to be identified. RANSAC is done after to weed out potential outliers.

### Blending in Poor Lighting Conditions
When we applied our alpha blending on the images, we see many artifacts between images due to poorly lit areas in cropped image.

<pic src="images/poorblend.png" width="350" alt="Logo">Blending artifacts</pic>
<pic src="images/betterblend.png" width="300" alt="Logo">Reasonable "blending"</pic>

Even after cropping the images, we still have artifacts when blending between them. This is due to alpha blending between poorly lit areas in one photo to the corresponding area in the other photo that is well lit. To get more natural results, we decided to change our blending technique. Instead of alpha blending, we just choose the brighter pixel to get a reasonable result. Any tiny artifacts are not noticeable due to the messy nature of caves.

A potential improvement for this would be to find seams between the images that will maximize the brightness of the image. This will make the rock structures look more natural.

# Experiment 3

## Drift
<pic src="images/drift.png" width="600" alt="Logo">Drift</pic>

With our current setup, we were able to stitch a 360 panorama of the image. However, due to the difficulties in photography, the tilting of the camera result in drift in the stitch. This is different from the vertical distortion we fixed in our midterm update as the warp portions are not stretched. Rather they bend downwards, making it unusable for cylindrical mapping.

<pic src="images/bent.png" width="300" alt="Logo">Original Image</pic>
<pic src="images/bb.png" width="300" alt="Logo">Blue dots is bounding box, Red is target rectangle</pic>
<pic src="images/straighten.png" width="300" alt="Logo">Processed Image</pic>


To straighten this, we applied thefollowing algorithm at every stitch:
1. Find a bounding box over the stitched image
2. Calculate a rectangle to map the image to
3. Do perspective transform to fit bent image in rectangle

<pic src="images/cave.png" width="600" alt="Logo">After straightening</pic>

With this naiive adjustment, we were able to reduce the drift by a certain extent. However the underlying bend on the image was not resolved.

Due to time constraint, we were unable to fully automate the straightening of the image.

## GUI Display
As mentioned previously, we decided to do a cylindrical panorama. We used three.js, a graphics library to map our image to the inside of the sphere. In order to achieve this, we had to warp image into a rectangle so that it can be texture mapped. We also had to crop it so that the left and right image would seamlessly join at one end of the cylinder.

To checkout our GUI click [here]({{baseUrl}}/cave_panorama.html)

## Conclusion
### Current Pipeline
With these updates, our pipeline is a follows:
```

Crop and Apply cylindrical warp on first image
stitch image = first image
for image in subsequent images
  1. Crop Apply cylindrical warp on image
  2. Apply CLAHE illumination on both greyscale images
  3. Extract features from greyscale stitch image and image using SIFT
  4. Use KNN to match the features to identify corespondences
  5. Given correspondences, run RANSAC to find homography matrix
  6. Warp image onto reference frame of stitch image
  7. Do blending between current image and image by choosing brightest pixel
  8. Straighten image
  9. Set stitch image = merged image
```
### Deviations from proposal
Due to our difficulties with stitching, we decided to settle for a cylindrincal view for our GUI. We also did not manage to fully account for drift in our stitching and consequently could not automate a proccess to crop the right border for a seamless 360.

### References
* [Base Stitcher, OpenCV Code](https://www.pyimagesearch.com/2016/01/11/opencv-panorama-stitching)
* [How to Write a Panorama Stitcher](http://ppwwyyxx.com/2016/How-to-Write-a-Panorama-Stitcher/)
