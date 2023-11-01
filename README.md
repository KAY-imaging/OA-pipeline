
# OA-workflow
The provided Python script is a pipeline for processing and analyzing images of vascularization using the Thermo Fisher Scientific Amira / Avizo 3D Visualization & Analysis Software (2022.2). It applies a series of filters and processing techniques to enhance medical image data and extract  
meaningful information. It utilizes features provided by the Amira / Avizo software to perform these operations and save the results for further analysis.

Here we want to describe how our pipeline could be implemented to characterize 3D images of vascularisation.
---
***Image preprocessing***
>1. Nifti dimentions converter  
>  1.1. Uploads file from 'folder_path'  
>  1.2. Returns converted files to 'out_path' folder

The 'matlab_script_converter_nifti', 'folder_path' and 'out_path' need to be defined  
The script retrieves a list of files from a specified directory. 

It uploads files from 'folder_path', converts them using Nifti dimentions converter - a MATLAB script that allows file conversion into the .nii format. 

Upon conversion the script returns the converted files to 'out_path' folder.

>2. Paths and tags  
  2.1. Define the path to the folder with images  
  2.2. Select the files of interest by specifying the 'tag'  

During this step we can select the files of interest by specifying the 'tag', which should be a part of the file name. This function creates a new folder with the name specified by the tag parameter within the given folder path.

>3. Image preparation  
  3.1. Provide the path and tag of files to the Avizo console  
  3.2. Run the cell and paste the following text output into Avizo console 
 
After the images are sorted into a separate folder according to the tag, we can move to the Image preparation  which includes providing the path and tag of files to the Avizo console and ROI selection.

>4. ROI selection  
  4.1. Paste the following code into Avizo console

This step uploads the sorted images from the folder created in the previous step and performs Volume Rendering.

>5. Manual selection of ROI  
  5.1. Select ROI of your choice  
  5.2. Create mask using the 'Volume Edit' tool in Avizo

You can manually select the desired ROI, which will be cut from the image, and create a mask with the 'Volume Edit' tool. This mask will be saved in a separate folder in the next step:

>6. Saving  
  6.1. Paste the following code into Avizo console

The first part of this pipeline is then complete with clearing of the project view window in Avizo and prepare it for the next step:

>7. Clear 'Project view' in Avizo  
  7.1. Clear and restart the Python console  
  7.2. Copy the new_path and tag into the Avizo console again (see beginning of Step 2)

---

***Image processing*** 
>8. Process the image and save the results  
>    Warning: this step requires computer resources.  
>    We do not recommend to upload stacks of large images.  

>  8.1. For images with other extentions:  
>  8.2. Change the 'extention' variable  
>  8.3. Paste the following code into Avizo console  

>    Filter settings can be adjusted with respect to your image properties  
>    See the settings below  


extention = '.nii'              # File extention variable
                                # Can be changed for other file types

number_of_images_for_threshold = 0    # Set number of images in the series
                                      # (if number_of_images_for_threshold = 0, the threshold will be calculated for each image)

first_point_shift = 0.985    # The percentage by which the left autorange limit is moved
                             # Calculated as (default_autorange - default_autorange * first_point_shift)

second_point_shift = 0.9     # The percentage by which the right autorange limit is moved
                             # Calculated as (default_autorange - default_autorange * second_point_shift)

multiplier_for_frameshift_of_volren = 1  # Range frameshift for the Volume Rendering filter
                                         # If the image is too bright, adjust the settings respectively

threshold_multiplier = 2.5    # Threshold shift multiplier to avoid noise and mess
                              # Removes ranges from ranges frame that a visualized by threshold_multiplier
                              # Calculated as:
                              # ((first_point_shift/second_point_shift * ranges from autothreshold) * multiplier_for_frameshift_of_volren)) * threshold_multiplier

To avoid the common error of Avizo auto-threshold setting, we preform the shift of the frame to the histogram values that will allow for proper visualization.

---

For each image it applies a **Hessian filter** to enhance the image structure.  

We apply the Hessian filte to a 3D image, it is used to detect and highlight textural features and structures in volumetric data. In our case it is used to enhance the tubular and elongated structures of the blood vessels. It is based on the computation of the Hessian, which is a matrix of second partial derivatives of image intensity. In a 3D image, Hessian matrix is calculated from each point representing a volumetric voxel (similar to a pixel in 2D) of the input image. The Hessian filter can help identify volumetric textural features (such as rod-like vessels structure), reflecting the changes of the intensity in different directions around a specific point.
[Dirk-Jan Kroon (2023). Hessian based Frangi Vesselness filter (https://www.mathworks.com/matlabcentral/fileexchange/24409-hessian-based-frangi-vesselness-filter) ]. 

To vary Hessian filter settings you need to specify **'hessian_filter'** parameters according to comments in the python notebook for the series of NIfTI images you would like to process.
**MAX** and **MIN** are the maximum and minimum radius of the structure respectively (shown in voxels). **STD** (shown in voxels) represents the standart deviation of the measurement that we use to identify our object. It is preferable for the STD value to be > voxel size.  

---
Next, we apply the **unsharp mask filter** to further enhance image details.
This filter is applied to reduce the impact of noise and to preserve the fine structures that can be easily lost in subsequent stages of image processing. It works by applying a smoothing filter to the original image to create its blurred version. After that, the original image is subtracted from the blurred image to enhance the contrast and sharpness of the blood vessel edges. The edge size and the edge contrast must be adjusted in accordance with the image analyzed.

The filtered files are renamed to indicate the applied filters.

---
**Volume rendering** is further performed on the filtered files.

Ranges for intensity rendering are collected to determine appropriate rendering ranges for each file.

The volume rendering parameters are adjusted based on the autoranges.  

---
**Interactive thresholding** is applied to the filtered files using the adjusted ranges for threshold rendering thus separating desired structures from the background.  

Also we can vary the threshold mode to specify it automatically or related to root images used for threshold recalculation.

---
**Pruning filtering** and **Centrline tree** reconstruction are performed on the thresholded binary images.

**Pruning filter.** The filter processes the dataset and applies the specified criteria to determine which structures or objects should be removed. 
Unwanted structures or objects that fall below the specified criteria are eliminated from the dataset.

**Centerline tree reconstruction.** This tool is used for extracting and visualizing the central axes of the blood vessels. 

---
Statistics are collected using **Spatial Graph Statistics** on the **Centrline tree** calculations results.

---
The project is autosaved in **Amira/Avizo** format.

---
Attribute graphs generated by **Spatial Graph Statistics** are exported as XML files.

---




**Dataset / images:**

Package                  Version
---------------------------------
et-xmlfile 	 (>= 1.1.0)  
matplotlib 	 (>= 3.6.2)  
numpy 		   (>=1.23.4)  
openpyxl 	   (>= 3.1.2)  
pandas  	   (>= 1.5.2)  
scipy 		   (>= 1.10.0)  
seaborn	     (>= 0.12.1)  
Statannot 	 (>= 0.2.3)  
XlsxWriter 	 (>= 3.1.3)  
Xmltodict 	 (>= 0.13.0)  
```
#!pip install et-xmlfile==1.1.0
#!pip install matplotlib==3.6.2
#!pip install numpy==1.23.4
#!pip install openpyxl==3.1.2
#!pip install pandas==1.5.2
#!pip install scipy==1.10.0
#!pip install seaborn==0.12.1
#!pip install Statannot==0.2.3
#!pip install XlsxWriter==3.1.3
#!pip install Xmltodict==0.13.0
```

