# Windows Installation Guide: liboqs & liboqs-python  

This guide provides a streamlined approach to compiling the C-based `liboqs` library and linking it to Python on Windows.

### Prerequisites
Before starting, ensure you have the following installed:
* **Python 3.x** (Ensure "Add Python to PATH" was checked during installation)
* **Git** ---

### Step 1: Install the C++ Build Environment

We need a C++ compiler and CMake.

1. Download **Build Tools for Visual Studio 2026** from the official Microsoft download page (scroll to the bottom, under "All Downloads" -> "Tools for Visual Studio").
   * https://visualstudio.microsoft.com/downloads   
   * https://aka.ms/vs/stable/vs_BuildTools.exe   
2. Run the installer (`vs_BuildTools.exe`).
3. Under the **Workloads** tab, check **ONLY** one option:
   * **Desktop development with C++**
4. On the right-side **Installation details** panel, uncheck unnecessary components to save space, but **ensure these three are checked**:
   * MSVC v14x - VS C++ x64/x86 build tools
   * Windows 11 SDK
   * C++ CMake tools for Windows
5. Click **Install**. You can close the installer once it finishes.

---

### Step 2: Compile liboqs (v0.14.0) as a Dynamic Library

Python requires a dynamic library (`.dll`) to interact with C code. We must compile `liboqs` specifically to generate this file.

1. Click the Windows Start menu, search for **"Developer Command Prompt for VS"**, right-click it, and select **Run as administrator**.
   *(Note: Do not use standard cmd or PowerShell. The Developer Prompt automatically configures the C compiler environment).*
2. **Important:** The prompt defaults to `C:\Windows\System32`. Do not clone code here. Move to a safe root directory:
   ```cmd
   cd /d C:\
   mkdir dev
   cd dev
   ```
3. Run the following commands sequentially to clone and compile version 0.14.0:
   ```cmd
   :: 1. Clone the specific version
   git clone --branch 0.14.0 https://github.com/open-quantum-safe/liboqs.git
   cd liboqs

   :: 2. Configure CMake (Must include -DBUILD_SHARED_LIBS=ON to generate the .dll)
   cmake -S . -B build -DBUILD_SHARED_LIBS=ON

   :: 3. Build the project
   cmake --build build --config Release

   :: 4. Install to the default Windows system directory
   cmake --install build
   ```
   *The compiled files are now located in `C:\Program Files (x86)\liboqs`.*

---

### Step 3: Configure Windows Environment Variables

Python needs to know where the newly compiled `.dll` file is located.

1. Open Windows Search and type **"Edit the system environment variables"**, then hit Enter.
2. Click the **Environment Variables...** button at the bottom right.
3. Under the **System variables** section (the bottom list), find and double-click the variable named **`Path`**.
4. Click **New** and paste the exact path to the `bin` folder:
   ```text
   C:\Program Files (x86)\liboqs\bin
   ```
5. Click **OK** on all three windows to save the settings.

---

### Step 4: Install liboqs-python (v0.14.0)

For the new environment variables to take effect, you **must completely close any open terminals or IDEs (like VS Code)** before proceeding.

1. Open a fresh, standard Command Prompt (`cmd`) or your Python virtual environment.
2. Install the exact matching version of the Python wrapper:
   ```cmd
   pip install liboqs-python==0.14.0
   ```

---

### Step 5: Verify the Installation

Create a test script (e.g., `test_pqc.py`) and run the following code to ensure the Python wrapper is successfully communicating with the underlying C library.

```python
import oqs

def test_environment():
    c_lib_version = oqs.oqs_version()
    print(f"Underlying liboqs (C) version: {c_lib_version}")

    if c_lib_version != "0.14.0":
        print("Warning: Loaded version is not 0.14.0. Check your Environment Variables.")
        return

    # Test a standard Post-Quantum algorithm (e.g., ML-KEM-768)
    kem_name = "ML-KEM-768"
    supported_kems = oqs.get_enabled_KEM_mechanisms()
    
    if kem_name in supported_kems:
        with oqs.KeyEncapsulation(kem_name) as client:
            public_key = client.generate_keypair()
            print(f"{kem_name} keypair generated successfully!")
            print(f"Public Key Length: {len(public_key)} bytes")
    else:
        print(f"{kem_name} is not available in this build.")

if __name__ == "__main__":
    test_environment()
```
