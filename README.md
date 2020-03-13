# IVUS Segmentation
Executables for segmenting lumen and plaque in IVUS image. (ver.1)<br/>


## Environments
Windows 8.1 (64bit), Python 2.7.12, Anaconda 4.1.1, CUDA 7.5, cuDNN 4.0.7, [caffe-windows](https://github.com/BVLC/caffe/tree/windows) <br/>
<br/>
<br/>

## Trained weights
FCN-all-at-once-VGG16 was proposed as a training model for LV 2D segmentation, and the scheme of the model is shown below. Each trained weights at displacements 0, 1, and 2 are in the [*weights*](./weights) directory.<br/>
<img src="https://user-images.githubusercontent.com/17020746/72951565-32a01580-3dd2-11ea-9011-598a490f30a4.png" width="100%">
[*weights/IVUS_SEG_00/snapshot.caffemodel*](./weights/IVUS_SEG_00)<br/>
[*weights/IVUS_SEG_01/snapshot.caffemodel*](./weights/IVUS_SEG_01)<br/>
[*weights/IVUS_SEG_02/snapshot.caffemodel*](./weights/IVUS_SEG_02)<br/>
<br/>
<br/>

## Lumen Segmentation
#### [Step1] Preparation of Data
*Preprocess.exe* is an executable file that downsamples 512x512 sized images into 256x256 sized images, which are learning input sizes.
The inputs are 'input file path' and 'output file path'. <br/>
The following code is an example of a command to downsample the "data\image.mha" image and save it as a "logs\downsampled.mha" image.
```DOS.bat
release\Preprocess.exe data\image.mha logs\image_downsampled.mha
```
*IVUS_displacement_make_RGB.exe* is an executable file that merges (i)th frame (i+N)th frame, and (i-N)th frame information into an RGB image.
N is hereinafter referred to as displacement. *IVUS_displacement_make_RGB.exe* uploaded to this repository outputs the images at displacement = 0,1,2 
of the input image all at once.<br/>
The following code is a command to convert the "logs\downsampled.mha"image to "logs\dipl_RGB_00_seg.mha", "logs\dipl_RGB_01_seg.mha", 
and "logs\dipl_RGB_02_seg.mha" images for displacement = 0,1,2 respectively.
```DOS.bat
release\IVUS_displacement_make_RGB.exe logs\image_downsampled.mha logs\displ
```

#### [Step2] Segmentation with trained weights
Run *MIRL_IVUS_Seg.py* to get the segmentation result. 
In *MIRL_IVUS_Seg.py*, a file specifying the model sturcture (*deploy.prototxt*) and a weights file (*snapshot.caffemodel*) 
are loaded to generate a segmentation mask from the input image.<br/>
The following code is a command to output the segmentation mask result ("logs\displ_RGB_01_seg.mha") of the "logs\displ_RGB_01.mha" image from the weights in "weights\IVUS_SEG_01".
```DOS.bat
python "%cd%\MIRL_IVUS_Seg.py" "%cd%\weights\IVUS_SEG_01" "%cd%\logs\displ_RGB_01.mha" "%cd%\logs\displ_RGB_01_seg.mha"
```
Then, the segmentation masks at different displacement are output as the final mask by *MajorityVoting.exe*. <br/>
The following code is an example of the command to extract the final mask into the "logs\mask.mha" file. 
Since the prefix is "logs\displ", the files "logs\displ_RGB_00.mha", "logs\displ_RGB_01.mha", and "logs\displ_RGB_02.mha" must all exist.
```DOS.bat
release\MajorityVoting.exe logs\displ logs\mask.mha
```

#### [Step3] Restoration to original form
There is no tremendous process. It just simply upsamples the results of size 256x256 to size 512x512.
*Postprocessing.exe* receives 'reference file path', 'file path to upsample', 'upsampled file path', and 'csv file recording voxel counts'. 
In the 'csv file recording voxel counts' file, the mask pixels in each frame are counted and recorded.
```DOS.bat
release\Postprocessing.exe data\image.mha logs\mask.mha result\mask.mha "logs\VoxelCount.csv"
```
