# OpenCV

## Docker

La imagen docker a utilizar será una modificación de [schickling/opencv](https://hub.docker.com/r/schickling/opencv)  que incluye CMake. El Dockerfile es el siguiente:

```dockerfile
FROM debian:jessie

ARG OPENCV_VERSION="3.0.0"

# install dependencies
RUN apt-get update
RUN apt-get install -y libopencv-dev yasm libjpeg-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libdc1394-22-dev libv4l-dev python-dev python-numpy libtbb-dev libqt4-dev libgtk2.0-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev x264 v4l-utils pkg-config
RUN apt-get install -y curl build-essential checkinstall cmake

# download opencv
RUN curl -sL https://github.com/Itseez/opencv/archive/$OPENCV_VERSION.tar.gz | tar xvz -C /tmp
RUN mkdir -p /tmp/opencv-$OPENCV_VERSION/build

WORKDIR /tmp/opencv-$OPENCV_VERSION/build

# install
RUN cmake -DWITH_FFMPEG=OFF -DWITH_OPENEXR=OFF -DBUILD_TIFF=OFF -DWITH_CUDA=OFF -DWITH_NVCUVID=OFF -DBUILD_PNG=OFF ..
RUN make
RUN make install

# configure
RUN echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf
RUN ldconfig
RUN ln /dev/null /dev/raw1394 # hide warning - http://stackoverflow.com/questions/12689304/ctypes-error-libdc1394-error-failed-to-initialize-libdc1394

# cleanup package manager
RUN apt-get remove --purge -y curl build-essential checkinstall
RUN apt-get autoclean && apt-get clean
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# prepare dir
RUN mkdir /source

VOLUME ["/source"]
WORKDIR /source
CMD ["bash"]
```
Se crea una carpeta y se crea el archivo `Dockerfile`.

```bash
mkdir Dockerfile
mv Dockerfile
```
Luego se construye el Dockerfile ejecutando:

```bash
$ docker build --tag dcnls/opencv .
```

### Uso

```bash
$ docker run --rm -it -v $(pwd):/source --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" --volume="$XAUTHORITY:$XAUTHORITY" --env="XAUTHORITY" dcnls/opencv
```

### Compilar archivo simple

```bash
$ g++ $(pkg-config --cflags --libs opencv) my-file.cpp
```
### Compilar archivo con CMake

Crear archivo `CMakeLists.txt` , copiar y rellenar `<NOMBRE PROYECTO>` en las siguientes líneas.

```bash
cmake_minimum_required(VERSION 2.8)
project( <NOMBRE PROYECTO> )
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_executable( <NOMBRE PROYECTO> <NOMBRE PROYECTO>.cpp )
target_link_libraries( <NOMBRE PROYECTO> ${OpenCV_LIBS} )
```

Cambiar al directorio del proyecto y ejecutar la compilación.

```bash
cd <PROYECTO_directory>
cmake .
make
```
