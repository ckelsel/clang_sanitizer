#build

```
mkdir build
cd build
CC=clang CXX=clang cmake -DCMAKE_BUILD_TYPE=Debug ..
make
./app 2>&1 | ../asan_symbolize.py | c++filt 

```
