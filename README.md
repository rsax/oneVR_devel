# stitchVR
Project started from an assignment in a Computational Photography class that stitches a set of input images together to create a realistic panorama.
<p>
Scale Invariant Feature Transform (SIFT) feature detection code from `http://www.cs.ubc.ca/~lowe/keypoints/`
<p>
Plans
-----
0. Planning to port everything to Python/NumPy for continued development since MATLAB is proprietary. Want to eventually release a free Virtual Reality (VR) video stitcher for amateur VR photographers as current stitching packages are expensive.
1. Currently only works with still photographs; will need to extend to stereoscopic video.
2. Current projection method is done using inverse cylindrical coordinates; want to cover a full 4π steradian FOV so may transition to using a spherical projection or cubic projection if beneficial. Seems like cubic projection is computationally more efficient but may result in undesired degradation of image quality.

Brief Usage
-----------
The scripts currently work with all files in the same directory and stitches together a set of `/input_images`.
<p>
Only intended adjustable user parameter is the focal length `f` in `main.c` which depends on the camera that generated the `/input_images`.

General Algorithm
-----------------

0. Acquire images from `/input_images`
0. Warp the `input_images` using inverse cylindrical coordinates (equations in `cylindrical.pdf`) to a helper directory `/cylindrical_maps`.
0. Run SIFT (Scale Invariant Feature Transform) on cylindrical projections to detect feature points between all pairs of adjacent images.
0. Using a RANSAC (Random Sample Concensus) approach, randomly select 4 pairs of feature points to generate a homography (3x3 matrix transform) relating each pair of adjacent images. Test the homography on other pairs of feature points between the 2 adjacent images and make note if the homography is within a specified accuracy tolerance of 3 pixels. Repeat for 100 iterations, choosing another 4 random pairs of feature points, and return the homography that is most consistent among all the feature points. Code is in `calcH.m`
0. Using the first image as a reference plane for the panorama, use the RANSAC generated homographies to transform the remaining input images to the appropriate pixels in the end panorama. Feather the overlapping regions with a linear opacity falloff from center of overlapping images. Code is found in `feather_mask.m` and `main.c`.
0. Output final panorama as `pano.jpg`

<h4>Bugs:</h4>
Artifacts seen in `pano_cyl_artifacts.jpg` at input image borders due to deficiencies in blending when using cylindrical projections.
<p>
Artifacts disappear when using input images without cylindrical projection as seen in `pano.jpg` but this method results in unwanted distortions for wide FOV panoramas.
<p>
Evident in sample input, some images overlap an area with a non-directly adjacent image, need to modify feathering technique to account for this.
<p>
Also, for some reason, script only outputs composite of first 4 images. May be a limitiation of MATLAB function `imwrite()`. Will look into this at a later time.
