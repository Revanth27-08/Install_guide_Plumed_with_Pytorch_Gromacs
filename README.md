# Install_guide_Plumed_with_Pytorch_Gromacs

Guide on compiling Plumed with support to Pytorch and Gromacs.

For further info, visit:
	https://www.plumed.org/doc-master/user-doc/html/_p_y_t_o_r_c_h.html
	https://github.com/luigibonati/mlcvs
	
-----------------

PLUMED

- load the latest gcc and openmpi in your machine, or install them through
	sudo apt install gcc openmpi-bin libopenmpi-dev
Also you will need	
	sudo apt install cmake git libopenblas-dev liblapack-dev 

- download the last version of Plumed with
	git clone --recursive -b master https://github.com/plumed/plumed2.git plumed2-master

- enter the folder and create this config_pytorch.sh file and run it

	LIBTORCH=${PWD}/libtorch
	if [ ! -d "$LIBTORCH" ]; then
	  wget https://download.pytorch.org/libtorch/lts/1.8/cpu/libtorch-cxx11-abi-shared-with-deps-1.8.2%2Bcpu.zip
	  unzip libtorch-cxx11-abi-shared-with-deps-1.8.2+cpu.zip
	  rm libtorch-cxx11-abi-shared-with-deps-1.8.2+cpu.zip
	  echo "export CPATH=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:$CPATH" >> ${LIBTORCH}/sourceme.sh
	  echo "export INCLUDE=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:$INCLUDE" >> ${LIBTORCH}/sourceme.sh
	  echo "export LIBRARY_PATH=${LIBTORCH}/lib:$LIBRARY_PATH" >> ${LIBTORCH}/sourceme.sh
	  echo "export LD_LIBRARY_PATH=${LIBTORCH}/lib:$LD_LIBRARY_PATH" >> ${LIBTORCH}/sourceme.sh
	fi
	. ${LIBTORCH}/sourceme.sh
	source ${LIBTORCH}/sourceme.sh
	./configure --enable-libtorch --enable-modules=all --disable-external-lapack --disable-external-blas --enable-mpi

- check in the output of the config_pytorch.sh if libtorch is enabled (checking libtorch with -lc10... yes) and mpi is enabled (PLUMED will be compiled using MPI). These two are the most problematic usually.
If not, try loading another compiler and/or openmpi version. 
On HPC resources, you might encouter further trouble with the compiler. A solution is to disable the ABI libraries.
In case, send me an email at valeriorizzi33@gmail.com and ask me. 

- when successful, in the terminal run
	source libtorch/sourceme.sh
	make
so that plumed is compiled and finally run
	source sourceme.sh
	
-----------------

GROMACS

- Download the latest version that can be patched with Plumed and decompress the code
	wget ftp://ftp.gromacs.org/pub/gromacs/gromacs-2022.3.tar.gz
	tar -zxvf gromacs-2022.3.tar.gz
	cd gromacs-2022.3

- patch plumed and choose the appropriate version of gromacs in the menu
	plumed patch -p --runtime

- create the folders for making the code and installing it
	mkdir build_single_cuda
	mkdir install_single_cuda
	cd build_single_cuda

- For a gpu cuda installation, you need to have the proper developer driver with theh Cuda toolkit.
In Ubuntu 20.04, it can be found here
	https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local
follow the guide and run this in your terminal
	export PATH="/usr/local/cuda-X.X/bin:$PATH"
with X.X being your cuda version.

- configure the installation with 
	cmake .. -DGMX_BUILD_OWN_FFTW=ON -DGMX_MPI=on -DGMX_OPENMP=ON -DGMX_GPU=CUDA -DCMAKE_INSTALL_PREFIX=/path-to-your-folder/gromacs-2022.3/install_single_cuda
If, for example, you want to install it on cpu only and use double precision
	cmake .. -DGMX_BUILD_OWN_FFTW=ON -DGMX_MPI=on -DGMX_OPENMP=ON -DGMX_GPU=OFF -DGMX_DOUBLE=on -DCMAKE_INSTALL_PREFIX=/path-to-your-folder/gromacs-2022.3/install_double_cpu


- compile and install
	make
	make install

-----------------

Extra tips

- to use PLUMED and GROMACS, add to your ~/.bashrc
	source /path-to-your-folder/plumed2-master/sourceme.sh
	source /path-to-your-folder/plumed2-master/libtorch/sourceme.sh    
	export PATH="/usr/local/cuda-X.X/bin:$PATH"
	source /path-to-your-folder/gromacs-2022.3/install_single_cuda/bin/GMXRC  

- if you have multiple versions, you can load them at will through alias such as
	alias splumed='source /path-to-your-folder/plumed2-master/sourceme.sh; source ...'
