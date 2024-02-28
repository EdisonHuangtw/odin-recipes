## Contour Detection 

1. **Method Description.** 

   - ***Warm up.***: The goal of this step is to modify the convolution process to minimize artifacts at image boundaries. We specify boundary='symm' to handle boundaries symmetrically. This means that the input is extended by reflecting the values at the boundaries, mirroring the image. 

   - ***Smoothing.***: Applying derivative of Gaussian filters helps obtain more robust estimates of the gradient, reducing sensitivity to noise in the image. Here, I found that using a sigma value of 2 yields better performance.

   - ***Non-maximum Suppression.***: Implement non-maximum suppression to produce thin edges. This involves looking at the gradient magnitude at the two neighbors in the direction perpendicular to the edge and suppressing the output at the current pixel if it is not greater than at the neighbors. Use sin/cos to find the y and x real coordinate and use bilinear interpolation to estimate the value.


2. **Precision Recall Plot.** 
   <div align="center">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/plot.png" width="60%">
   </div>


3. **Results Table.** 

   | Method                                          | overall max F-score | average max F-score | AP       | Runtime (seconds) |
   | ----------------------------------------------- | ------------------- | ------------------- | -------- | ----------------- |
   | Initial implementation                          | 0.450               | 0.516               | 0.385    | 0.010             |
   | Warm-up [remove boundary artifacts]             | 0.496245            | 0.555469            | 0.468797 | 0.013698          |
   | Smoothing                                       | 0.594689            | 0.632555            | 0.583030 | 0.021321          |
   | Non-maximum Suppression                         | 0.601189            | 0.641194            | 0.600768 | 1.293390          |
   | Val set numbers of best model [From gradescope] | 0.601193            | 0.642216            | 0.603742 | 1.340852          |

4. **Visualizations.** In the context of the red square space, employing Gaussian filtering can effectively diminish noise while potentially sacrificing some level of detail. Additionally, implementing Non-Maximum Suppression (NMS) can lead to the generation of thinner lines. In the purple square space, there are still numerous detected edges, possibly due to the challenging task of distinguishing the grass area or the snow area. This difficulty arises from several factors, for example, the texture and color variability of grass and the presence of fine details in grass blades and foliage. These complexities can make it more challenging for edge detection algorithms to accurately delineate edges within grass areas compared to tree areas.

   <div align="center">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part1/bench/54082.png" width="30%" style="margin:10px;">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part2/bench/54082.png" width="30%" style="margin:10px;">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part3/bench/54082.png" width="30%" style="margin:10px;">

      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part1/bench/19021.png" width="30%" style="margin:10px;">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part2/bench/19021.png" width="30%" style="margin:10px;">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part3/bench/19021.png" width="30%" style="margin:10px;">

      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part1/bench/97033.png" width="30%" style="margin:10px;">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part2/bench/97033.png" width="30%" style="margin:10px;">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/contours/output/part3/bench/97033.png" width="30%" style="margin:10px;">
   </div>

5. **Bells and Whistles.** I tried to add hysteresis thresholding to my code. Hysteresis thresholding helps produce stronger and more continuous edges by considering both strong and weak edge pixels. Additionally, I attempted to use a multi-scale edge detector; however, the results seem unsatisfactory, with performance even decreasing.

   
   | Method                                | overall max F-score | average max F-score | AP       | Runtime (seconds) |
   | ------------------------------------- | ------------------- | ------------------- | -------- | ----------------- |
   | Best base Implementation (from above) | 0.601189            | 0.641194            | 0.600768 | 1.293390          |
   | Bells and whistle (1) [extra credit]) | 0.601193            | 0.642216            | 0.603742 | 1.340852          |
   | Bells and whistle (2) [extra credit]) | 0.579224            | 0.617648            | 0.583105 | 6.088318          |
   | Bells and whistle (n) [extra credit]) |                     |                     |          |
## Corner Detection 

1. **Method Description.** 

   - ***Compute Partial Derivatives at Each Pixel.***: Compute the partial derivatives of the image at each pixel to obtain the gradient information with the filter [-1 0 1] and its transpose.

   - ***Compute Second Moment Matrix M in a Gaussian Window.***: The second moment matrix is computed using the gradients obtained in the previous step. Additionally, Gaussian filtering is applied to reduce noise. Here, I found that using a sigma value of 0.7 yields the best performance.

   - ***Compute corner response function R.***: Use the eigenvalues to calculate the corner response function R with the function: R = det(M) - alpha * trace(M)^2

   - ***Find local maxima of response function.***: Unlike in contour detection, non-maximum suppression used in corner detection creates a window around each pixel and checks if the center pixel value is the largest. If not, it is set to 0. I found that setting the window size to 3 achieves the best performance.

2. **Precision Recall Plot.** 

   <div align="center">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/corners/plot.png" width="60%">
   </div>

3. **Results Table.** 
   | Method                                          | Average Precision | Runtime  |
   | ----------------------------------------------- | ----------------- | -------- |
   | Random                                          | 0.002             | 0.001    |
   | Harris w/o NMS                                  | 0.246328          | 0.002413 |
   | Harris w/ NMS                                   | 0.511119          | 0.157574 |
   | Hyper-parameters alpha - 0.05                   | 0.503364          | 0.168604 |
   | Hyper-parameters alpha - 0.06                   | 0.499970          | 0.163661 |
   | Val set numbers of best model [From gradescope] | 0.511119          | 0.123257 |


4. **Visualizations.**: The corner detection primarily identifies building corners well; however, it also misidentifies other objects, such as tree leaves, as corners. This occurs because tree leaves share texture and contrast similarities with corners, leading to high responses in corner detection algorithms. Additionally, the arrangement and shape of tree leaves can resemble corners, particularly from specific perspectives or lighting conditions.
   <div align="center">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/corners/output/demo/vis/draw_cube_17_vis.png" width="100%">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/corners/output/demo/vis/ece_vis.png" width="100%">
      <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/corners/output/demo/vis/beckman_vis.png" width="100%">
   </div>

5. **Bells and Whistles.** 

   | Method                                | Average Precision | Runtime  |
   | ------------------------------------- | ----------------- | -------- |
   | Best base Implementation (from above) | 0.511119          | 0.123257 |
   | Bells and whistle (1) [extra credit]) |                   |          |
   | Bells and whistle (2) [extra credit]) |                   |          |
   | Bells and whistle (n) [extra credit]) |                   |          |
## Multi-resolution Blending 

1. **Method Description.**
    
    - ***Image and Mask Stacking***:The image_stack function computes Laplacian pyramids for the input images (im1 and im2) by iteratively applying Gaussian filtering and subtracting the blurred images from the original images. Similarly, the mask_stack function computes Gaussian pyramids for the input mask by applying Gaussian filtering iteratively.

    - ***Combining Pyramids***:The combine_pyramid function combines the Laplacian pyramids of the images (LA and LB) using the Gaussian pyramid of the mask (GR) as weights. This step ensures that regions where the mask has high values are primarily influenced by the Laplacian pyramid of im1, while regions with low mask values are influenced by the Laplacian pyramid of im2.

    - ***Blending***:The blend function sums up the levels of the combined Laplacian pyramid (LS) to obtain the final blended image (out). This step effectively blends the two input images (im1 and im2) based on the provided mask, producing a seamless transition between them.

2. **Oraple.** 

<div align="center">
      <img src="blend/orange.jpeg" width="30%" style="margin:10px;">
      <img src="blend/apple.jpeg" width="30%" style="margin:10px;">
      <img src="blend/result.png" width="30%" style="margin:10px;">
</div>

3. **Blends of your choice.**

 I tried to combine the day and night images of the Eiffel Tower, where I had to crop the images and generate the mask myself.

<div align="center">
   ![moon](https://github.com/EdisonHuangtw/odin-recipes/assets/136220779/1bc0983e-1c82-4817-845f-4cfbe398552f)
    <img src="/home/edison/UIUC/ECE549/cv-sp24-mps/MP3/blend/paris/output.png" width="30%" style="margin:10px;">
</div>
