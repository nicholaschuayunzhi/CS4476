<frontmatter>
  title: "Proposal"
  footer: footer.md
</frontmatter>

<include src="nav.md" boilerplate />

# Approach
## General Process Pipeline
1. Caver takes multiple photos
1. Caver uses software to stitch images together
1. Software automatically stitches images into one giant image
    * The image should have sufficient information to wrap around in the x, y and z directions
        * If it does not have enough information then the non-captured areas will just be blacked-out
    * Blending and image correction is done by the software to account for issues with lighting in the cave
1. Software then maps 360 image into 3d space for viewing
    * Caver can view this image preferably in a GUI

# Experiment and Results

## Dataset:
We will be generating our own dataset for this project, as there does not openly exist a dataset of 360 degree cave images. In order to build this dataset, we will collect pictures from many different locations in some caves.
* For each location, the cameraman should not move the origin of the camera (only rotate)
* Number of pictures taken should roughly cover the full 360 area of the cave

## Exploited Code:
* [OpenCV](https://opencv.org/)
  * implementation of [base stitcher](https://www.pyimagesearch.com/2016/01/11/opencv-panorama-stitching/)

## Implemented Code:
* Cylindrical Warp
* Alpha Blending
* Multi-image 3D Mapping
* Lighting Filter
* GUI for input and display

## Success Criteria:
The project will be successful if it generates 3D images from cave interior pictures, with the images having lighting artifacts removed, and thus generating a clear 360 image.

## Experiments

**Experiment 1:** Stitch 2 Images together without manually setting correspondecnes
1. Manually select images to stitch together
1. Use SIFT to identify correspondences (OpenCV)
1. Image stitching using homographies
1. Blend images together with different blending techniques

**Experiment 2:** Stitch all images from cave location into large image
1. SIFT to identify correspondences
1. Use k-nn model to find matching features for each feature in the images
1. Use RANSAC to find homographies between images with these features
1. Generate the full cave image

**Experiment 3:** Conversion of stitched images into 3D space
1. Mapping of images to spherical/cylindrical coordinates
1. Apply final stitched image to sphere/cylinder via texture mapping

**Experiment 4:** Image correction due to cave lighting issues
1. We will work with both single images and the 3D stitched images to look at what works for fixing the lighting
1. Take shots with varying brightness to do flash reflection correction
1. Remove artifacts from noise or insufficient image data
    * Image inpainting
    * Texture creation with Markov Models

## Experiment Results
These experiments should allow us to build up to the final result of the project, an application that can take in many images from a cave, stitch them together into a 3D space, and apply some filters to fix lighting issues that arise when taking pictures in caves with non-professional photography equipment.

The first 3 experiments are image manipulation in physical space. As caves tend to have many unique and unusual features we should be able to form decent mappings of features, but **Experiment 1** will focus on making sure this is the case. Another focus of **Experiment 1** would be to test out various blending techniques and compare them with one another.

**Experiment 2** will follow this trend and check how well the feature mapping works over the entire cave space.

**Experiment 3** will determine both whether or not cylindrical or spherical projections work better for this task, as well as making sure that the feature mapping still works when not applied to a flat plane.

The major unknown with these experiments is whether or not the feature selection will work well, though we are fairly confident that it will.

**Experiment 4** will work with finding  the best filters and processing that needs to be done to the images to remove lighting issues and create clearer images.

---
