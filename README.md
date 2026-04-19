# liboqs & liboqs-python Installation Guide:   

This guide provides instructions for installing and compiling the `liboqs` and `liboqs-python` libraries on Windows and Raspberry Pi.



## For Windows 11


### Prerequisites
Before starting, ensure you have the following installed:
* **Python 3.10+** (Ensure "Add Python to PATH" was checked during installation)
* **Git** 


### Step 1: Install the C++ Build Environment

We need a C++ compiler and CMake.

1. Download **Build Tools for Visual Studio 2026** from the official Microsoft download page (scroll to the bottom, under "All Downloads" -> "Tools for Visual Studio").
   * Visual Studio 2026 Download Page: https://visualstudio.microsoft.com/downloads   
   * Directly Download the Build Tools: https://aka.ms/vs/stable/vs_BuildTools.exe   
2. Run the installer (`vs_BuildTools.exe`).
3. Under the **Workloads** tab, check **ONLY** one option:
   * **Desktop development with C++**
4. On the right-side **Installation details** panel, uncheck unnecessary components to save space, but ensure these three are checked:
   * **MSVC v14x - VS C++ x64/x86 build tools**
   * **Windows 11 SDK**
   * **C++ CMake tools for Windows**
5. Click **Install**. You can close the installer once it finishes.


### Step 2: Compile liboqs (v0.14.0)  

Python requires a dynamic library (`.dll`) to interact with C code. We must compile `liboqs` specifically to generate this file.

1. Click the Windows Start menu, search for **"Developer Command Prompt for VS"**, right-click it, and select **Run as administrator**.    
2. The prompt defaults to `C:\Windows\System32`. Do not clone code here. Move to a safe root directory:
   ```cmd
   cd /d C:\
   mkdir dev
   cd dev
   ```
3. Run the following commands sequentially to clone and compile:
   ```cmd
   :: 1. Clone the 0.14.0 version:   
   git clone --branch 0.14.0 https://github.com/open-quantum-safe/liboqs.git
   cd liboqs

   :: 2. Configure CMake:   
   cmake -S . -B build -DBUILD_SHARED_LIBS=ON

   :: 3. Build the project:   
   cmake --build build --config Release

   :: 4. Install to the default Windows system directory:  
   cmake --install build
   ```
   *The compiled files are now located in `C:\Program Files (x86)\liboqs`.*


### Step 3: Configure Windows Environment Variables

Python needs to know where the compiled `.dll` file is located.

1. Open Windows Search and type **"Edit the system environment variables"**, then hit Enter.
2. Click the **Environment Variables** button at the bottom right.
3. Under the **System variables** section (the bottom list), find and double-click the variable named **`Path`**.
4. Click **New** and paste the exact path to the `bin` folder:
   ```text
   C:\Program Files (x86)\liboqs\bin
   ```
5. Click **OK** on all three windows to save the settings.


### Step 4: Install liboqs-python (v0.14.1)

For the new environment variables to take effect, you must completely close any open terminals or IDEs before proceeding.  

1. Open a fresh, standard Command Prompt or your Python virtual environment.
2. Install the Python wrapper:
   ```cmd
   pip install liboqs-python
   ```


### Step 5: Verify the Installation

Create a test script (e.g., `test_pqc.py`) and run the following code to ensure the Python wrapper is successfully communicating with the underlying C library.

```python
import oqs
print("liboqs version:", oqs.oqs_version())
print("liboqs-python version:", oqs.oqs_python_version())
print("KEM supported in liboqs:")
print(oqs.get_supported_kem_mechanisms()[:3])
```
---
---


## For Raspberry Pi


### Step 1: Prepare the System and Base Dependencies

This step ensures we have the correct build tools (`cmake`, `ninja`) and the essential development libraries required for cryptographic algorithms (OpenSSL headers).

```bash
# 1. Update the system package list  
sudo apt update
sudo apt upgrade -y

# 2. Install build tools and dependencies   
sudo apt install -y build-essential cmake ninja-build libssl-dev git python3-pip python3-venv
```


### Step 2: Compile and Install liboqs (v0.14.0)
`liboqs-python` is just a wrapper; it heavily relies on the `.so` dynamic library existing in the system. We must compile the C library correctly first.

```bash
# 1. Return to the home directory
cd ~

# 2. Clone the specific 0.14.0 version source code
git clone --branch 0.14.0 https://github.com/open-quantum-safe/liboqs.git
cd liboqs

# 3. Create a build directory
mkdir build && cd build

# 4. Configure the project using CMake  
cmake -GNinja -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=/usr/local ..

# 5. Compile and install to the system path
ninja
sudo ninja install

# 6. Refresh the system's shared library cache    
sudo ldconfig
```  
Verification: Run `ls -l /usr/local/lib/liboqs*` to ensure you can see `liboqs.so.0.14.0`.


### Step 3: Configure the Python Virtual Environment
To avoid polluting the Raspberry Pi's system-level Python environment, we use a virtual environment for isolation.

```bash
# 1. Return to the home directory
cd ~

# 2. Create a virtual environment named oqs_env
python3 -m venv oqs_env

# 3. Activate the virtual environment 
# (Once successful, your command line prefix will show "(oqs_env)")
source oqs_env/bin/activate

# 4. Update base packages (Optional but recommended)
pip install --upgrade pip setuptools
```


### Step 4: Install liboqs-python
You must compile and install from the local source code while the virtual environment is **activated**. This ensures it perfectly binds to the 0.14.0 C library you just compiled.

```bash
# 0. Ensure the virtual environment is ACTIVATED
source oqs_env/bin/activate

# 1. Return to the home directory
cd ~

# 2. Clone the Python wrapper source code
git clone https://github.com/open-quantum-safe/liboqs-python.git
cd liboqs-python

# 3. Compile and install locally   
pip install . --no-cache-dir
```

---

### Step 5: Final Verification
Run the following Python one-liner directly in the terminal to test your installation:

```bash
python3 -c "import oqs; print('\n Installation Successful!\nCurrent liboqs and liboqs-python version:', oqs.oqs_version(), oqs.oqs_python_version()); print('Supported KEM algorithms:', oqs.get_enabled_kem_mechanisms()[:3])"
```

---

Updated on April 19, 2026