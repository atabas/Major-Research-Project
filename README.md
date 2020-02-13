
## Developing a Confidence Measure Based Evaluation Metric for Breast Cancer Screening Using Bayesian Neural Networks
#### Major Research Project for MSc Data Science and Analytics at Ryerson University 
 
 ## Get the Dataset
 
 Download the dataset from the [Cancer Imaging Archive](https://wiki.cancerimagingarchive.net/display/Public/CBIS-DDSM#5e40bd1f79d64f04b40cac57ceca9272).
 Download `Mass-Training ROI and Cropped Images (DICOM)` and put it in your working directory. Put training images
 under a folder called `Mass-Full-Images_CBIS-DDSM/Training-cropped` and test images under a folder called
 `Mass-Full-Images_CBIS-DDSM/Test-cropped`. Then run the code as instructed below.
 
 ## Instructions to run:

 The code is divided into 3 main segments represented by 3 different jupyter notebook files. To run everything end to end, 
 follow the steps below for each of the notebooks in the given order. You will need to have certain python packages installed 
 in your python virtual environment as mentioned in the import statements. Also, you will need to have a GPU and cuda installed
 if you don't run it in Google Colab.
 
 ### Training and Validation Data Generation (file: [hdf5_generation.ipynb](https://github.com/atabas/Major-Research-Project/blob/master/hdf5_generation.ipynb))
 
 This file contains the code for generating the training and validation datasets. Since processsing dicom files on the
 fly is pretty expensive, we have decided to generate an HDF5 file that will contain all the necessary data (images) and labels
 to make training much faster.
 
 - Run the notebook step by step in your python virtual environemnt (except for the very last cell of the notebook). We are parsing 
 the dicom files to extract the actual image data along with their labels and resizing the images to 224 * 224
 - This should generate a file called `ddsm_cropped_all.hdf5` in your working directory. The structure of this file is such that
 it contains 10 placeholders for our 5-fold cross validation split. Every set of 2 placeholders is for the training and validation
 data for each split.
 - The last of the notebook file shows exploratory analysis regarding the class distribution.
 
 ### Training (file: [ddsm_experiment_cropped_cv.ipynb](https://github.com/atabas/Major-Research-Project/blob/master/ddsm_experiment_cropped_cv.ipynb))
 
 This file contains the bulk of the code for non-Bayesian and Bayesian training and classification. We have done our 
 experiment on Google Colab, so there's some Colab-specific code here. However, you can just try this notebook on your own 
 environemnt (by changing the file paths) in the same working directory where the `ddsm_cropped_all.hdf5` file was generated, 
 given that you have enough computing power.
 The code flow for this notebook file is detailed as:
 
 - A custom PyTorch dataset class is created and 2 data loaders are defined for training and validation.
 - ResNet-18 is used and earlier layers are frozen and a new fully connected block is configured.
 - Using the image files in the hdf5 file, non-Bayesian training is performed with random augmentation applied on the images
 - After reaching a reasonable accuracy, the fuly connected block is taken out and the convolutional block is used as a feature
 generator to generate 512-element vectors.
 - With augmentation applied, 10-fold (relative to the original) amount of data (512-element feature vectors) is generated
 for the Bayesian training. Validation data is not augmented at all.
 - The Bayesian neural network with just the 3 fully connected layer is initialized and random normal priors are assigned to the
 weights and biases.
 - Using Pyro's built-in API, the Gaussian Process (SVI) is performed and posteriors are inferred.
 - From the posterior, 1000 networks are sampled.
 - Results (accuracy and coverage) are generated for N and P ranging from 0.2 to 0.95, with an interval of 0.05 between each value, producing 256 combinations of N and P values. These are saved into `tuning_results.csv`.
 
 ### Evaluation (file: [result_analysis.ipynb](https://github.com/atabas/Major-Research-Project/blob/master/result_analysis.ipynb))
In this file, we analyze the results saved in `tuning_results.csv`. First, N is tuned for a fixed canonical value (0.5) of P and then P is tuned for the same value of N. We demonstrate that in both cases (increasing value of N at a fixed P or increasing value of P at a fixed N), accuracy increases and coverage decreases. These results are also plotted in graph format.

&copy;Anika Tabassum
