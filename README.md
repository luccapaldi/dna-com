# dna-com
Visualize λ-DNA center of mass based on pixel intensities of tiff image stacks obtained using an Andor camera.

Images here...

This script computes the center-of-mass of a λ-DNA trapped within a nanofluidic cavity over time. This project was inspired by the analysis completed in Section 3.2 of the following publication:

**Capaldi, X., Liu, Z., Zhang, Y., Zeng, L., Reyes-Lamothe, R., &amp; Reisner, W. (2019). Correction: Probing the organization and dynamics of two DNA chains trapped in a nanofluidic cavity. Soft Matter, 15(42), 8639–8639. https://doi.org/10.1039/c9sm90211b**

Special thanks to [Walter Reisner](http://www.physics.mcgill.ca/~reisner/)'s research group at McGill University for providing the raw data, which consists of a tiff image stack and corresponding camera metadata. In addition, the script makes use of an external function from [tondu](https://github.com/xcapaldi/tondu), also authored by members of the Reisner Lab, to compute the timestep associated with each frame. Note that the data must be pre-processed using an algorithm developed by Patrick Doyle's research group at MIT. I have implemented a version here: [doyle-background-subtraction](https://github.com/luccapaldi/doyle-background-subtraction). If you use this work, please make sure to cite all relevent publications.

The script reads the pre-processed data into a multi-dimensional Numpy array. The location of the center of mass for each frame is calculated based on the average of the pixel intensity. For each frame, the coordinates of the center-of-mass and the frame timestep are used to calculate the displacement and instantaneous velocity of the DNA. Since a single channel of the original fluorescence microscopy data was provided, the grayscale image is converted to RGB by duplicating the pixel intensity across all three channels (resulting in a 'black-and-white' image). The pixel corresponding to the center-of-mass for each frame is converted to red and the image stack is displayed as a looping video. In addition, the insantaneous velocities are decomposed into x-coordinates (due to surface roughness) and y-coordinates (due to the flow) and plotted as histograms.

To run this script, you would need to ensure that numpy, tifffile, scipy, pickle, and matplotlib are installed and that [tondu](https://github.com/xcapaldi/tondu) was in the same directory. However, since this script is intended as a demonstration, the raw data files are not provided. If you are interested in utilizing or expanding this work, I encourage you to reach out to me directly.
    





