# TEASER++
[<img src="https://github.com/MIT-SPARK/TEASER-plusplus/workflows/build/badge.svg">](https://github.com/MIT-SPARK/TEASER-plusplus/actions)

TEASER++ is a fast and certifiably-robust point cloud registration framework written in C++. It can solve the rigid body transformation problem between two point clouds in 3D. TEASER++ also has bindings for MATLAB and Python, as well as integration with ROS for easy testing and visualization.   

![](doc/banner.png)
*Left: correspondences generated by [3DSmoothNet](https://github.com/zgojcic/3DSmoothNet) (green and red lines represent the inlier and outlier correspondences according to the ground truth respectively). Right: alignment estimated by TEASER++ (green dots represent inliers found by TEASER++).*

For a short conceptual introduction, check out our [video](https://www.youtube.com/watch?v=xib1RSUoeeQ). For more information, please refer to our paper:
- [H. Yang](http://hankyang.mit.edu/), [J. Shi](http://jingnanshi.com/), and [L. Carlone](http://lucacarlone.mit.edu/), "TEASER: Fast and Certifiable Point-Cloud Registration,". [arXiv:2001.07715](https://arxiv.org/abs/2001.07715) [cs, math], Jan. 2020. ([pdf](https://arxiv.org/pdf/2001.07715.pdf))

Other related publications from our group include:
- [H. Yang](http://hankyang.mit.edu/) and [L. Carlone](http://lucacarlone.mit.edu/), “A Polynomial-time Solution for Robust Registration with Extreme Outlier Rates,” in Robotics: Science and Systems (RSS), 2019. ([pdf](https://arxiv.org/pdf/1903.08588.pdf))
- [H. Yang](http://hankyang.mit.edu/) and [L. Carlone](http://lucacarlone.mit.edu/), “A quaternion-based certifiably optimal solution to the Wahba problem with outliers,” in Proceedings of the IEEE International Conference on Computer Vision (ICCV), 2019, pp. 1665–1674. ([pdf](https://arxiv.org/pdf/1905.12536.pdf))
- [H. Yang](http://hankyang.mit.edu/), [P. Antonante](http://www.mit.edu/~antonap/), [V. Tzoumas](https://vasileiostzoumas.com/), and [L. Carlone](http://lucacarlone.mit.edu/), “Graduated Non-Convexity for Robust Spatial Perception: From Non-Minimal Solvers to Global Outlier Rejection,” IEEE Robotics and Automation Letters (RA-L), 2020. ([pdf](https://arxiv.org/pdf/1909.08605))

If you find this library helpful or use it in your project, please consider citing:
```bibtex
@article{Yang20arXiv-TEASER,
    title={TEASER: Fast and Certifiable Point Cloud Registration},
    author={Yang, Heng and Shi, Jingnan and Carlone, Luca},
    year={2020},
    eprint={2001.07715},
    archivePrefix={arXiv},
    primaryClass={cs.RO},
    url = {https://github.com/MIT-SPARK/TEASER-plusplus},
    pdf = {https://arxiv.org/abs/2001.07715}
}
```

If you are interested in more works from us, please visit our lab page [here](http://web.mit.edu/sparklab/).

## Getting Started
### Supported Platforms

TEASER++ has been tested on Ubuntu 18.04 with g++-7/9 and clang++-7/8/9.  

### Dependencies
Building TEASER++ requires the following libraries installed: 
1. Eigen3 >= 3.3
3. PCL >= 1.9 (optional)
4. Boost >= 1.58 (optional)

You may run the following to install Eigen3:
```shell script
sudo apt install libeigen3-dev
```
Please refer to other dependencies' documentation for detailed installation instructions. 

You also need a compiler that supports OpenMP. See [here](https://www.openmp.org/resources/openmp-compilers-tools/) for a list. 

If you want to build Python bindings, you also need:
1. Python 2 or 3 (make sure to include the desired interpreter in your `PATH` variable)

If you want to build MATLAB bindings, you also need:
1. MATLAB 
2. CMake >= 3.13

TEASER++ uses the Parallel Maximum Clique ([paper](https://arxiv.org/abs/1302.6256), [code](https://github.com/ryanrossi/pmc)) for maximum clique calculation. It will be downloaded automatically during CMake configuration. In addition, CMake will also download Google Test and pybind11 if necessary.

### Compilation and Installation
Ensure that your CMake version is at least 3.10. Clone the repo to your local directory. Open a terminal in the repo root directory. Run the following commands:
```shell
mkdir build
cd build
cmake ..
make
```
If you want to install relevant headers and shared libraries, run `make install` after the above commands (you may need to `sudo`).

### Available CMake Options

| Option Name            | Description         | Default Value |
|------------------------|---------------------|---------------|
|`BUILD_TESTS`             | Build tests         |  ON           |
|`BUILD_TEASER_FPFH`       | Build TEASER++ wrappers for PCL FPFH estimation | OFF |
|`BUILD_MATLAB_BINDINGS`   | Build MATLAB bindings | OFF |
|`BUILD_PYTHON_BINDINGS`  | Build Python bindings | ON |
|`BUILD_DOC` | Build documentation   | ON |
|`BUILD_WITH_MKL`| Build Eigen with MKL  |  OFF|
|`BUILD_WITH_MARCH_NATIVE`| Build with flag `march=native` | OFF |
|`ENABLE_DIAGNOSTIC_PRINT`| Enable printing of diagnostic messages | OFF |

### Run Tests
You can run the tests by running `ctest` in the `build` directory. To generate the Doxygen documentation, run `make doc`, and you should be able to view the Doxygen files in `build/doc` folder. 

By default, the library is built in release mode. If you instead choose to build it in debug mode, some tests are likely to time out. 

To run benchmarks (for speed & accuracy tests), you can execute the following command:
```shell
ctest --verbose -R RegistrationBenchmark.*
```
The `--verbose` option allows you to see the output, as well as the summary tables generated by each benchmark.

## How to use TEASER++
To use TEASER++ in your CMake-based C++ project, here's an example CMake file that finds and includes TEASER++:
```cmake
cmake_minimum_required(VERSION 3.10)
project(teaserpp_example)

set (CMAKE_CXX_STANDARD 14)

find_package(Eigen3 REQUIRED) 
find_package(teaserpp REQUIRED)

# Change this line to include your own executable file
add_executable(cpp_example cpp_example.cpp)

# Link to teaserpp & Eigen3 
target_link_libraries(cpp_example Eigen3::Eigen teaserpp::teaser_registration)
```

Here's a short C++ snippet that you may find helpful for integrating TEASER++ in your code:
```c++
#include <Eigen/Core>
#include <teaser/registration.h>

Eigen::Matrix<double, 3, Eigen::Dynamic> src(3, N);
Eigen::Matrix<double, 3, Eigen::Dynamic> dst(3, N);

// Populate src & dst with your correspondences ...

// Populate solver parameters
teaser::RobustRegistrationSolver::Params params;
params.cbar2 = 1;
params.noise_bound = 0.01;
params.estimate_scaling = false;
params.rotation_estimation_algorithm = teaser::RobustRegistrationSolver::ROTATION_ESTIMATION_ALGORITHM::GNC_TLS;
params.rotation_gnc_factor = 1.4;
params.rotation_max_iterations = 100;
params.rotation_cost_threshold = 1e-6;

// Initialize solver
teaser::RobustRegistrationSolver solver(params);

// Solve
solver.solve(src, dst);

// Get solution
auto solution = solver.getSolution();
```

For a short example on how to use the MATLAB bindings for TEASER++, please refer to [this](matlab/README.md) document.

For a short example on how to use the Python bindings for TEASER++, please refer to [this](python/README.md) document.

To use TEASER++ in a ROS environment, simple clone the repo to your catkin workspace.

## Known Issues
- If you are encountering segmentation faults from PMC, try add the environmental variable `OMP_NUM_THREADS=12` in your current shell. You can also just prepend `OMP_NUM_THREADS=12` when running your executable.
- When using the MATLAB wrapper with MATLAB on terminal (`-nojvm` option enabled), you might encounter errors similar to this:
`/usr/local/MATLAB/R2019a/bin/glnxa64/MATLAB: symbol lookup error: /opt/intel/compilers_and_libraries_2019.4.243/linux/mkl/lib/intel64_lin/libmkl_vml_avx2.so: undefined symbol: mkl_serv_getenv`. One way to get around this is to run the following command in the environment where you start MATLAB: 
`export LD_PRELOAD=/opt/intel/mkl/lib/intel64/libmkl_intel_lp64.so:/opt/intel/mkl/lib/intel64/libmkl_gnu_thread.so:/opt/intel/mkl/lib/intel64/libmkl_core.so`. You may need to change the paths according to your MKL installation.
