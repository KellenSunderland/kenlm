# This Dockerfile creates a linux library for KenLM that is designed to be as portable as possible.
# This .so is intended to be bundled into a .jar file and published on maven for Java developers via 
# a Java API which invokes the .so through JNI.

FROM centos:centos6.8

# Install required pre-reqs.
RUN yum update -y
RUN yum install -y cmake git gcc gcc-c++ zlib-devel wget zlib-static

# Copy KenLM source
RUN mkdir -p /opt/kenlm
WORKDIR /opt/kenlm
COPY . /opt/kenlm

# Define JAVA_HOME
ENV JAVA_VERSION 8u111
ENV BUILD_VERSION b14
ENV JAVA_HOME /usr/java/jdk1.8.0_111/

# Define other flags
ENV KENLM_MAX_ORDER 10
ENV CXXFLAGS "$CXXFLAGS -fPIC -O3 -DHAVE_ZLIB"
ENV CMAKE_CXX_FLAGS "$CXXFLAGS -fPIC -O3 -DHAVE_ZLIB"
ENV LDFLAGS "$LDFLAGS -lz -lrt -static-libstdc++ -static-libgcc -Bstatic -lbz2 -Bdynamic"

# Update GCC to 4.9
RUN yum install -y centos-release-scl && \
    yum install -y devtoolset-3-toolchain
RUN /usr/bin/scl enable devtoolset-3 true

# Download and install Java 
RUN wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" \
 "http://download.oracle.com/otn-pub/java/jdk/$JAVA_VERSION-$BUILD_VERSION/jdk-$JAVA_VERSION-linux-x64.rpm" -O /tmp/jdk-8-linux-x64.rpm
RUN yum -y install /tmp/jdk-8-linux-x64.rpm

# Setup bzip2
RUN mkdir -p /opt/bzip2
WORKDIR /opt/bzip2
RUN wget http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz && \
    tar -xvzf bzip2-1.0.6.tar.gz && \
    cd bzip2-1.0.6 && \
    scl enable devtoolset-3 bash && \
    make install

# Setup Boost
RUN mkdir -p /opt/boost
WORKDIR /opt/boost
ENV BOOST_ROOT /opt/boost/boost_1_62_0
RUN wget http://downloads.sourceforge.net/project/boost/boost/1.62.0/boost_1_62_0.tar.gz && \
    tar -xvzf boost_1_62_0.tar.gz
WORKDIR /opt/boost/boost_1_62_0
RUN ./bootstrap.sh && \
    ./b2 install -j 8; exit 0

# Build KenLM
WORKDIR /opt/kenlm
RUN rm -rf build && mkdir build && cd build &&  \
    scl enable devtoolset-3 "cmake ..    -DKENLM_MAX_ORDER=$KENLM_MAX_ORDER -DFORCE_STATIC=On" && \
    scl enable devtoolset-3 "make kenlm -j 8"

# Build KenLM wrapper (.so with JNI exports)
RUN scl enable devtoolset-3 "g++ $CXXFLAGS -std=c++0x \
         -I. -DKENLM_MAX_ORDER=$KENLM_MAX_ORDER \
         -I$JAVA_HOME/include \
         -I$JAVA_HOME/include/linux \
         -I$JAVA_HOME/include/darwin \
         /opt/kenlm/java/kenlm_wrap.cc \
         /opt/kenlm/build/lib/*.a \
         -shared -o libken.so -static-libstdc++ -static-libgcc" 

# # Define default command.
CMD ["bash"]