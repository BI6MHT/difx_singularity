# difx_singularity

参考文档：

[DiFX Installation](https://www.atnf.csiro.au/vlbi/dokuwiki/doku.php/difx/installation)

[CentOS 安装 DiFX](https://shaoguangleo.github.io/2018/01/24/astronomy-difx-install-on-centos/)


## Linux依赖库安装

```
yum install deltarpm -y # 一种优化 RPM 软件包管理的机制，它主要用于减小软件包更新时的数据传输量。
yum upgrade -y # 更新时把旧的版本删除
yum install vim wget make
yum install gcc gcc-c++ gcc-gfortran kernel-devel python3 # mpich4.0以上的版本需要python3


yum install pkgconfig bison  
yum install expat expat-devel expat-static
yum install subversion doxygen
yum install automake autoconf
yum install libtool* flex*
yum install systemd-devel

yum install libXt-devel libXext-devel # X服务

yum install portmap # 端口映射
```

## IPP的安装

```
yum-config-manager --add-repo https://yum.repos.intel.com/ipp/setup/intel-ipp.repo
# 导入 Intel 软件产品的 GPG 公钥以进行 RPM 包验证，可能出现下载超时
wget https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB
rpm --import GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB 
yum install intel-ipp-64bit-2018.2-046.x86_64

vim ~/.bashrc
# ADD IPP
export IPPROOT=/opt/intel/
source ~/.bashrc
```

## MPI的安装

```
# 可以选择openmpi或者mpich，也可以尝试不同的版本，修改和mpi相关的路径后，重新编译difx就可以了

## mpich-4.1.2的安装（二选一）
mkdir /ASTRO
mkdir /ASTRO/PACKAGES/INSTALL
cd /ASTRO/PACKAGES/INSTALL
wget https://www.mpich.org/static/downloads/4.1.2/mpich-4.1.2.tar.gz
tar -xzvf mpich-4.1.2.tar.gz
cd mpich-4.1.2
mkdir /ASTRO/PACKAGES/mpich-4.1.2
./configure --prefix=/ASTRO/PACKAGES/mpich-4.1.2
make
make install

vim ~/.bashrc
# ADD MPI
export MPIROOT=/ASTRO/PACKAGES/mpich-4.1.2
export PATH=$MPIROOT/bin:$PATH
export LD_LIBRARY_PATH=$MPIROOT/lib:$LD_LIBRARY_PATH
export MANPATH=$MPIROOT/man:$MANPATH


## openmpi-1.6.5的安装（二选一）
mkdir /ASTRO
mkdir /ASTRO/PACKAGES/INSTALL
cd /ASTRO/PACKAGES/INSTALL
wget https://download.open-mpi.org/release/open-mpi/v1.6/openmpi-1.6.5.tar.gz
tar -xzvf openmpi-1.6.5.tar.gz
cd openmpi-1.6.5
mkdir /ASTRO/PACKAGES/openmpi-1.6.5
./configure --prefix=/ASTRO/PACKAGES/openmpi-1.6.5 --enable-mpi-cxx
make
make install

vim ~/.bashrc
# ADD MPI
export MPIROOT=/ASTRO/PACKAGES/openmpi-1.6.5
export PATH=$MPIROOT/bin:$PATH
export LD_LIBRARY_PATH=$MPIROOT/lib:$LD_LIBRARY_PATH
export MANPATH=$MPIROOT/man:$MANPATH
source ~/.bashrc
```

## 安装pgplot

```
cd /ASTRO/PACKAGES/INSTALL
wget ftp://ftp.astro.caltech.edu/pub/pgplot/pgplot5.2.tar.gz
tar -zxvf pgplot5.2.tar.gz
mkdir /ASTRO/PACKAGES/pgplot-5.2
cp pgplot/drivers.list /ASTRO/PACKAGES/pgplot-5.2/
cd /ASTRO/PACKAGES/pgplot-5.2/

/ASTRO/PACKAGES/INSTALL/pgplot/makemake /ASTRO/PACKAGES/INSTALL/pgplot linux g77_gcc

vim ~/.bashrc
# ADD PGPLOT
export PGPLOTROOT=/ASTRO/PACKAGES/pgplot-5.2
export PATH=$PGPLOTROOT:$PATH
source ~/.bashrc
```

# 安装fftw

```
wget http://www.fftw.org/fftw-3.3.8.tar.gz
tar -xvf fftw-3.3.8.tar.gz
cd fftw-3.3.8
mkdir /ASTRO/PACKAGES/fftw-3.3.8
# 编译两次，获得不同的精度的动态库lib
./configure --prefix=/ASTRO/PACKAGES/fftw-3.3.8  --enable-mpi --enable-shared=yes
make
make install
./configure --prefix=/ASTRO/PACKAGES/fftw-3.3.8  --enable-float  --enable-mpi --enable-shared=yes
make
make install

vim ~/.bashrc
# ADD FFTW
export FFTWROOT=/ASTRO/PACKAGES/fftw-3.3.8
export LD_LIBRARY_PATH=$FFTWROOT/lib:$LD_LIBRARY_PATH
export PKG_CONFIG_PATH=$FFTWROOT/lib/pkgconfig:$PKG_CONFIG_PATH

source ~/.bashrc
```

# DiFX安装

```
cd /ASTRO/PACKAGES/INSTALL
svn co https://svn.atnf.csiro.au/difx/master_tags/DiFX-2.5.5 --username youremail
cd DiFX-2.5.5/
mkdir /ASTRO/DiFX-2.5.5
vim setup.bash
        ####### DIFX VERSION ########################
        export DIFX_VERSION=DiFX-2.5.5

        ####### ROOT PATHS ##########################
        export DIFXROOT=/ASTRO/DiFX-2.5.5
        export DIFX_PREFIX=$DIFXROOT
        export PGPLOTDIR=$PGPLOTROOT
        export IPPROOT=$IPPTOOT

        ####### COMPILER ############################
        export DIFXMPIDIR=$MPIROOT
        export MPICXX="${DIFXMPIDIR}"/bin/mpicxx
source setup.bash
python3 genipppc $IPPROOT/
mkdir -p  $DIFXROOT/lib/pkgconfig/
cp ipp.pc $DIFXROOT/lib/pkgconfig/
./install-difx --mk5daemon --withm6support --withmark6meta

find / -name calcserver # 找到calcserver所在位置
cp ./applications/calcserver/init.d/calcserver /etc/init.d/
```
