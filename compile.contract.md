# 编译安装

- 环境
    Ubuntu 16.04 LTS

## 1. 安装 clang 4.0

```bash
apt update && apt -y install software-properties-common
apt-add-repository -y "deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-4.0 main" && apt update
apt -y install clang-4.0 lldb-4.0 libclang-4.0-dev wget make python-dev libbz2-dev libdb++-dev libdb-dev libssl-dev openssl libreadline-dev autoconf libtool git ntp doxygen libc++-dev cmake  g++ libcurl4-openssl-dev unixodbc-dev libjsoncpp-dev uuid-dev build-essential vim
add-apt-repository -y ppa:ubuntu-toolchain-r/test && apt update
apt -y install libstdc++-7-dev
```

## 3. 安装cmake

```bash
mkdir -p /opt/cmake
mkdir -p ~/software && cd ~/software
wget https://cmake.org/files/v3.11/cmake-3.11.0-Linux-x86_64.sh
chmod +x cmake-3.11.0-Linux-x86_64.sh
bash cmake-3.11.0-Linux-x86_64.sh --prefix=/opt/cmake --skip-license
ln -sfT /opt/cmake/bin/cmake /usr/local/bin/cmake
```

## 4. 安装 boost 1.67

```bash
mkdir -p ~/software && cd ~/software
wget https://dl.bintray.com/boostorg/release/1.67.0/source/boost_1_67_0.tar.gz -O  boost_1_67_0.tar.gz
tar -zxvf boost_1_67_0.tar.gz && cd boost_1_67_0 && chmod +x bootstrap.sh
./bootstrap.sh --prefix=/usr
./b2 --buildtype=complete install
```

## 5. 安装 llvm with wasm

```bash
mkdir -p ~/software && cd ~/software
mkdir  wasm-compiler && cd wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git

#tar xzvf wasm-compiler.tar.gz

cd ~/software/wasm-compiler/llvm
mkdir -p build && cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/opt/wasm -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ..
make -j4 install
```

## 6. 编译

```bash
mkdir -p ~/software && cd ~/software
git clone https://github.com/my-graphene/core
cd core
git submodule update --init --recursive


echo "" >> ~/.bashrc
echo "export WASM_ROOT=/opt/wasm" >> ~/.bashrc
echo "export C_COMPILER=clang-4.0" >> ~/.bashrc
echo "export CXX_COMPILER=clang++-4.0" >> ~/.bashrc
echo "" >> ~/.bashrc

source ~/.bashrc

mkdir -p build &&  cd build
cmake -DWASM_ROOT=${WASM_ROOT} -DCMAKE_CXX_COMPILER="${CXX_COMPILER}" -DCMAKE_C_COMPILER="${C_COMPILER}" -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_BUILD_TYPE=Debug ..
make -j4 install
```
