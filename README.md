# 5 Minutes to Buck

This is a very quick Buck tutorial. The final source-files are above. 

> 🚨 This tutorial was written with Linux in mind. Windows and macOS users, please refer to [Facebook's setup instructions](https://buckbuild.com/setup/getting_started.html). Otherwise, the steps are the same. 

## 0. What tools will I need?

You will need GCC / Clang, a text editor and Buck. 

You probably have the first two, so here's how to install Buck for Linux:

```
wget https://github.com/njlr/buck-warp/releases/download/v0.3.0/buck-2019.01.10.01-linux -O buck
chmod +x ./buck
sudo mv ./buck /usr/local/bin/buck
buck --version
```

## 1. How do I create an executable?

Create the files we need:

```
touch .buckconfig
touch BUCK
touch main.cpp
```

`main.cpp` will contain a hello-world application:

```cpp
#include <iostream>

using namespace std;

int main() {
  cout << "Hello, world. " << endl;

  return 0;
}
```

`BUCK` will describe how to build our application:

```python
cxx_binary(
  name = 'app',
  srcs = [
    'main.cpp',
  ],
)
```

Now you can run the application: 

```bash
buck run :app
```

You should see:

```bash
Hello, world. 
```

# 2. How do I create a library?

These instructions follow on from the previous. 

We will put our library into a folder called `math`. Create these files: 

```bash
mkdir math
touch ./math/BUCK
touch ./math/square.hpp
touch ./math/square.cpp
```

You project should look like this:

```bash
tree . 
.
├── BUCK
├── main.cpp
└── math
    ├── BUCK
    ├── square.cpp
    └── square.hpp

1 directory, 5 files
```

Now we need to fill write the contents. 

`math/square.hpp`

```cpp
#ifndef MATH_SQUARE_HPP
#define MATH_SQUARE_HPP

int square(int x);

#endif
```

`math/square.cpp`

```cpp
#include <math/square.hpp>

int square(int x) {
  return x * x;
}
```

`math/BUCK`: 

```python
cxx_library(
  name = 'math',
  exported_headers = [
    'square.hpp',
  ],
  srcs = [
    'square.cpp',
  ],
  visibility = [
    'PUBLIC',
  ],
)
```

Now we should be able to build the library: 

```bash
buck build //math:math
```

Success!

We can request static or shared flavors: 

```bash
# Shared
buck build //math:math#linux-x86_64,shared --show-output

# Static
buck build //math:math#linux-x86_64,static --show-output
```

The `--show-output` flag asks Buck where it put the file. In this case: 

```bash
ls buck-out/gen/math/math\#linux-x86_64,shared
ls buck-out/gen/math/math#linux-x86_64,static/
```

# 3. How do I use the library from the application?

First, we need to edit `BUCK` to tell Buck that `//:app` depends on `//math:math`:

```
cxx_binary(
  name = 'app',
  srcs = [
    'main.cpp',
  ],
  deps = [
    '//math:math',
  ],
)
```

Notice how we added a `deps` property. 

Now, we can change `main.cpp` to actually use the library: 

```cpp=
#include <iostream>
#include <math/square.hpp>

using namespace std;

int main() {
  cout << "Hello, world. " << endl;
  cout << "square(4) = " << square(4) << endl;

  return 0;
}
```

And for the big finale `buck run :app`...

```
$ buck run :app
Parsing buck files: finished in 0.7 sec
Building: finished in 0.8 sec (100%) 8/8 jobs, 2 updated
  Total time: 1.8 sec
Hello, world. 
square(4) = 16
```
