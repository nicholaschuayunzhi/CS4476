<frontmatter>
  title: "Midterm Update"
  footer: footer.md
</frontmatter>

<include src="nav.md" boilerplate />

# Midterm Update 10/31/2018
Currently we have completed experiment 1 and made good progress on experiment 2 on **non-cave** images.

<pic src="images/14merge.png" width="600" alt="Logo">14 Images Panorama</pic>


The initial set up uses a base stitcher class written in this [article](https://www.pyimagesearch.com/2016/01/11/opencv-panorama-stitching/
). It uses *OpenCV* to implement basic feature matching and stitching for two images. It generates images as follows:

```
1. Extract features from greyscale images using SIFT
2. Use KNN to match the features to identify corespondences
3. Given correspondences, run RANSAC to find homography matrix
4. Warp one image ontop of the other
```
We added support for stitching of multiple images and blending of the the images together.

# Experiment 1

### Alpha Blending

When stitching images together, it is common for consecutively taken images to have slightly different color distributions, which, when stitched together, leads to artifacts and line on the edges of the stitched images. We implemented alpha blending to rectify this issue. By using the expression:

```
final_Image =  α*warped_image + (1-α)*source_image
```

Where alpha is an array of the same size as the final image, containing values in [0 , 1], indicating where the overlap of the images are. By running a small gaussian filter over this image, we received an alpha like so:

<pic src="images/filter.png" width="300" alt="Logo">Gaussian Filter on Alpha Map</pic>
<pic src="images/badblend.png" width="300" alt="Logo">Without Alpha Blending</pic>
<pic src="images/goodblend.png" width="300" alt="Logo">With Alpha Blending</pic>

Where the edges of the image are the transitional values between 0 and 1. As these transitional values live around the edges of the stitched image, we receive a smooth transition from the colors of one image to another, with the pixels along the boundary being a linear combination of the source and warped image’s pixel values multiplied by the values of alpha and 1-alpha respectively. The main challenge with alpha blending is that sudden, large shifts in color/luminance between images are far more difficult to work with, needing a larger, smoother transition between 0 and 1 in alpha, while too large a transition can lead to ghosting artifacts. In our case, most of the images have very similar intensities, but a way to fix this is to have the transition between 0 and 1 be a function of the average intensity difference on either side of the stitched seam.

## Experiment 2

### Stitching Multiple Images
At first we use a naiive stitching algorithm where we stitch the first two images, and use that output for the next stitch. This is done from left to right. However, this results in vertical distortion:

<pic src="images/verticaldistort.png" width="400" alt="Logo">Vertical Distortion</pic>

The reason why this happens as explained in this [article](http://ppwwyyxx.com/2016/How-to-Write-a-Panorama-Stitcher/) is because a panorama is taken from a cylindrical/spherical lens instead of planar one. To account for this, we apply a cylindrical warp to all images before stitching.

<pic src="images/cylindricalwarp1.png" width="200" alt="Logo">Cylindrical Warp Left Image</pic>
<pic src="images/cylindricalwarp2.png" width="200" alt="Logo">Cylindrical Warp Right Image</pic>
<pic src="images/merged.png" width="400" alt="Logo">Merged Image</pic>

The formula we used for cylindrical warp is:

```
x' = focal_length * atan((x - xcenter) / focal_length)
y' = focal_length * (y - ycenter) / sqrt((x- xcenter)^2 + focal_length^2)
```

One key thing about the formula is that the measurements are all in pixel length. Thus we need to calculate the focal length of the camera in pixel length. The method we used to estimate this is:

```
focal_length_pixel = photo_width*(focal_length/CCD_width)
```
Here the focal length and CCD_width, the width of the camera sensor, are both in millimetres. We used the EXIF data of the photo to get the focal length and model of the camera. After which, we did a search on the model to find the camera sensor's width.

With this adjustment, we can minimize vertical distortion by a significant amount, resulting in a nicer stitch:
<pic src="images/verticaldistort.png" width="400" alt="Logo">4 Images without Cylindrical Warp</pic>
<pic src="images/4merge.png" width="400" alt="Logo">4 Images with Cylindrical Warp</pic>

Another important benefit of the warp is since the right end of the merged image is less warped, we are better able to find features and correspondences for additional images. Without the cylindrical warp, the algorithm could only go to 4 images. After the adjustment, we can go to 10 images before vertical errors show.

<pic src="images/longermerge.png" width="800" alt="Logo">10 Images with Cylindrical Warp</pic>

As can be seen from above, vertical errors still accumulate. We are currently experimenting with different ways on how we can reduce these errors. Things that we are trying include

* Merging small portions before doing a final merge:

<pic src="images/image0.jpg" width="300" alt="Logo">Images 0 to 5</pic>
<pic src="images/image1.jpg" width="300" alt="Logo">Images 6 to 11</pic>
<pic src="images/image2.jpg" width="300" alt="Logo">Images 11 to 17</pic>

* Adjusting focal length as we merge more images:
<pic src="images/longermerge.png" width="600" alt="Logo">No adjustment to warp</pic>
<pic src="images/11imagemergetuning.jpg" width="600" alt="Logo">Adjusting cylindrical warp as we merge more images</pic>

## Conclusion
### Current Pipeline
With these updates, our pipeline is a follows:
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
### Deviations from proposal
Currently we are working on non-cave images. We will have access to cave photos in November, taken by our team member.

### Future Work
We will work on fine tuning our stitching algorithm, allowing us to achieve panorama that will be able to wrap around.

Next we will look into mapping the panorama to the inside of the sphere which we will then display in a GUI.
We will also complement our panorama stitcher with image correction on the cave photos when we have obtained them, removing distortion and artifiacts caused by the cave's environment.


### References
* [Base Stitcher, OpenCV Code](https://www.pyimagesearch.com/2016/01/11/opencv-panorama-stitching)
* [How to Write a Panorama Stitcher](http://ppwwyyxx.com/2016/How-to-Write-a-Panorama-Stitcher/)
