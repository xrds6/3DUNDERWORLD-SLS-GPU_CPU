# 3DUNDERWORLD-Structured-Light-Scanner
[![status](http://joss.theoj.org/papers/4329bcbc7bba33961a5e749dcacb995b/status.svg)](http://joss.theoj.org/papers/4329bcbc7bba33961a5e749dcacb995b)
[![Build Status](https://travis-ci.org/theICTlab/3DUNDERWORLD-SLS-GPU_CPU.svg?branch=dev)](https://travis-ci.org/theICTlab/3DUNDERWORLD-SLS-GPU_CPU)

Structured light scanner is a tool to reconstruct point cloud from series of images lit by patterned light. As showing in the following image, patterns are projected to the object in the grid manner. Each cell in the grid has a unique sequence. The corresponding points can be extracted by decoding the sequences from both images.
![img](https://raw.githubusercontent.com/theICTlab/3DUNDERWORLD-SLS-GPU_CPU/dev/screenshots/flow.png)

In this version, the reconstruction process is reimplemented both on CPU and GPU. Both versions accelerate the reconstruction time. Especially the later one. 

## Dependencies
* OpenCV2
* CUDA (6.0+)
* glm

## Howto
### Compile and run demo binaries
Demo binaries and data is included.  A CMake script is included to compile the binaries and libraries. 
```bash
git clone https://github.com/theICTlab/3DUNDERWORLD-SLS-GPU_CPU.git
cd 3DUNDERWORLD-SLS-GPU_CPU
mkdir build
cd build
cmake ..
make
```
If CUDA is detected in your system, the cmake flag `ENABLE_CUDA` is set to `on` by default, both CPU and GPU constructors are created in the `bin` folder. Otherwise, only CPU constructor will be generated. To run the binaries, you can pass the data folder and camera configuration files as parameters, or using the default value if you compiled with the commands above.

You can use `cmake .. -DENABLE_CUDA=off` to disable CUDA if you don't want compile the GPU binary.

### Enable test
A test using [Google Test Framework](https://github.com/google/googletest.git) is included in the build. To use the test, enable `GTEST` flag at configuration stage.
```
git clone https://github.com/theICTlab/3DUNDERWORLD-SLS-GPU_CPU.git
cd 3DUNDERWORLD-SLS-GPU_CPU
mkdir build
cd build
cmake .. -DGTEST=ON
make
make test
```
Google test will be downloaded and compiled during the compilation stage. 

Please note that the test requires reference file from source path, so the CMake output folder needs to be placed in the source path. (Shown as the example above). 

### Run demo binary
```
usage: ./SLS [options] ... 
options:
  -l, --leftcam        Left camera image folder (string [=../../data/alexander/leftCam/dataset1/])
  -r, --rightcam       Right camera image folder (string [=../../data/alexander/rightCam/dataset1/])
  -L, --leftconfig     Left camera configuration file (string [=../../data/alexander/leftCam/calib/output/calib.xml])
  -R, --rightconfig    Right camera configuration file (string [=../../data/alexander/rightCam/calib/output/calib.xml])
  -o, --output         Right camera configuration file (string [=output.ply])
  -f, --format         suffix of image files, e.g. jpg (string [=jpg])
  -w, --width          Projector width (unsigned long [=1024])
  -h, --height         Projector height (unsigned long [=768])
  -?, --help           print this message

```
The configuration files are `xml` files generated by OpenCV calibration tool, including intrinsic and extrinsic parameters. 

### Grab the output
Most of the outputs are written to the log file named `SLS.log`. To track the outputs while running the binary, run `tail -f sls.log` in another terminal of the same directory. 

### Use the library

This project also provides libraries that can be easily integrated into other projects. Here's a quick start code for using the libraries.

```C++
#ifdef CPU

/* Using CPU reconstruction */
#include <core/fileReader.h>
#include <core/Reconstructor.h>
#else

/* Using CUDA reconstruction */
#include <ReconstructorCUDA/fileReaderCUDA.cuh>
#include <ReconstructorCUDA/ReconstructorCUDA.cuh>
#endif


int main()
{
    // Folders contain reconstruction images
    std::string rightCameraFolder = "../../data/alexander/rightCam/dataset1/"
    std::string leftCameraFolder = "../../data/alexander/leftCam/dataset1/"
    // Camera configuration files
    std::string rightConfigFile = "../../data/alexander/rightCam/calib/output/calib.xml"
    std::string leftConfigFile = "../../data/alexander/leftCam/calib/output/calib.xml"

    // Allocate two cameras from heap. Parameters are the names of cameras. 
#ifdef CPU
    SLS::FileReader *rightCam=new SLS::FileReader("rightCamera");
    SLS::FileReader *leftCam= new SLS::FileReader("leftCamera");
#else
    SLS::FileReaderCUDA *rightCam=new SLS::FileReaderCUDA("rightCamera");
    SLS::FileReaderCUDA *leftCam= new SLS::FileReaderCUDA("leftCamera");
#endif

    // Load image from data folder
    rightCam->loadImages(rightCameraFolder);
    leftCam->loadImages(leftCameraFolder);

    // Load configuration files
    rightCam->loadConfig(rightConfigFile);
    leftCam->loadConfig(leftConfigFile);

    // Initialize a reconstructor by passing the grid pattern size. 
#ifdef CPU
    SLS::ReconstructorCPU reconstructor(1024,768);
#else
    SLS::ReconstructorCUDA reconstructor(1024,768);
#endif
    // Pass cameras to the reconstructor.
    // Allocated resources will be released in constructor.
    reconstructor.addCamera(rightCam);
    reconstructor.addCamera(leftCam);
    reconstructor.reconstruct();

    // Export point cloud to file.
    SLS::exportOBJVec4(output,  reconstructor);
    return 0;
}
```

## Known issues

Since there's no good API for cameras, the camera acquisition is not implemented. However, interfaces are provided.
We welcome you to implement your camera class and make a pull request to this project.

Also please note that since the example data files are currently included in the repository, the size of repository is 300MB. It may take while to clone. 

## Version Information

### [v4] 
Lead software developer: Qing Gu
     
Researchers: Qing Gu, Charalambos Poullis
     
Immersive and Creative Technologies Lab (http://www.theICTlab.org), Concordia University

### [v1-v3.2] 

Lead software developer: Kyriakos Herakleous

Researchers: Kyriakos Herakleous, Charalambos Poullis

Immersive and Creative Technologies Lab (http://www.theICTlab.org), Concordia University


#### Part of the 3DUNDERWORLD project: http://www.3dunderworld.org

##### IMPORTANT: To use this software, YOU MUST CITE the following in any resulting publication:*

```
@article{GuHerakleousPoullis3dunderworld,
  title={A Structured-Light Scanning Software for Rapid Geometry Acquisition},
  author={Gu, Qing and Herakleous, Kyriakos and Poullis, Charalambos},
  journal={TBA},
  year={2016}
}

@article{DBLP:journals/corr/HerakleousP14,
  author    = {Kyriakos Herakleous and
               Charalambos Poullis},
  title     = {3DUNDERWORLD-SLS: An Open-Source Structured-Light Scanning System
               for Rapid Geometry Acquisition},
  journal   = {CoRR},
  volume    = {abs/1406.6595},
  year      = {2014},
  url       = {http://arxiv.org/abs/1406.6595},
  timestamp = {Tue, 01 Jul 2014 11:58:08 +0200},
  biburl    = {http://dblp.uni-trier.de/rec/bib/journals/corr/HerakleousP14},
  bibsource = {dblp computer science bibliography, http://dblp.org}
}
```

