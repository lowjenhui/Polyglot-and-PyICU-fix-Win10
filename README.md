# Polyglot-and-PyICU-fix-Win10
Instructions to get Polyglot, PyICU and Pycld2 to build on Windows 10.


1. You need **ICU**. There are quite a few ways of getting ICU onto your computer. Make sure you have the icu/bin in your path as well.
	- Anaconda **(This is the fastest method)**
		- `conda install -c conda-forge icu=58.1`
		- https://anaconda.org/conda-forge/icu
	- Download the Binaries and the Source from ICU's main site: http://site.icu-project.org/
		- If you are building from VS2015, make sure you have **updated it** with patches 1 & 2
		- Update it from "Tools > Extensions & Updates..."
		- Check the Readme.html on how to test whether it works inside the ICU file
	- Online precompiled binaries
		- https://www.npcglib.org/~stathis/blog/precompiled-icu/


2. **PyICU**. This is part needs more effort and there are a few things you need to change:
	- Download the 7z file for PyICU and try to build. `python setup.py build`

	This error here was an absolute NIGHTMARE:
  
          `ICU_VERSION = subprocess.check_output(('icu-config', '--version')).strip()` 
          `AttributeError: 'module' object has no attribute 'check_output'`
		
	- **To fix this, change that line in setup.py (in the PyICU folder) to:**
	  - `ICU_VERSION = subprocess.check_output(('icuinfo', '--version')).strip()`
		- Windows does not have 'icu-config', it's 'icuconfig'
	
	- Change the following lines and put in your icu path (E.g. 'C:/icu'):
		1. INCLUDES:
			`'win32': ['C:/icu/include'],`
		2. LFLAGS:
			`'win32': ['C:/icu/lib'],`
	
  
	- If you have more errors:
		- calendar.cpp error:
			- A very quick fix would be to comment out all lines that threw an error. There's something wrong with daylight. For me it was (362, 376, 381, 389)
			- This link here explains why: https://github.com/ovalhub/pyicu/issues/7
			- There is a fix here, but I didn't try it.
			
		- common.cpp error:
			- Replace: `u_memcpy(PyUnicode_2BYTE_DATA(result), utf16, len16);`
			- with: `u_memcpy((UChar *) PyUnicode_2BYTE_DATA(result), utf16, len16);`
			- This is to explicitly cast it
			
		- if you still have problems: There is a .whl you can use. Extract it in the folder which contains setup.py
		https://github.com/ovalhub/pyicu/files/829179/PyICU-1.9.5-cp36-cp36m-win_amd64.whl.zip

3. Pycld2:
	- You will most likely have problems installing this file if you updated and patched Visual Studios.
		- bindings\pycldmodule.cc(16): fatal error C1083: Cannot open include file: 'cstrings': No such file or directory
		- This is the <string.h> error:
			- In 'pycld2-0.31\bindings\pycldmodule.cc'
				- Replace <string.h> with <cstring> 
			- In 'pycld2-0.31\bindings\encodings.cc'
				- Replace <string.h> with <cstring> 
				- Replace `strcomp` with `_stricmp`
				


#### :sparkles: It should build successfully after this! :) :sparkles: 
