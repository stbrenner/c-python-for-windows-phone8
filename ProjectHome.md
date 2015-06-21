# python core for windows phone 8 #

Windows Phone 8 supports native code, which give us a choice to compile cpython code on windows phone 8 directly. But for some reason, some native apis for win32 are not supported on windows phone 8, such as native thread, loadlibrary, etc. Therefore, to compile cpython code for windows phone 8, some source code should be changed.

The project attempts to compile python 2.7.3 on wp8, and make it can be called by native code.

## modification of the python source code ##

**1.no longer support thread, **subprocess/threadmodule/thread.c are removed from the project**

_2.registry is not supported,_winreg is removed.**

**3.some others small changes**

# using python in native apps on windows phone 8 #

## 1. create project ##

open vs2012, create a windows phone 8 native project:

http://c-python-for-windows-phone8.googlecode.com/svn/wiki/images/example_1.JPG

## 2. add python27.dll ##

Add python27.dll to the project, set it’s property “content” to Yes.

http://c-python-for-windows-phone8.googlecode.com/svn/wiki/images/example_2.JPG

http://c-python-for-windows-phone8.googlecode.com/svn/wiki/images/example_3.JPG

## 3. set path ##

set include directores

http://c-python-for-windows-phone8.googlecode.com/svn/wiki/images/example_6.JPG

set library search path:

http://c-python-for-windows-phone8.googlecode.com/svn/wiki/images/example_4.JPG

## 4. Add python27.lib ##

http://c-python-for-windows-phone8.googlecode.com/svn/wiki/images/example_5.JPG

## 5. create pytest.py ##

Create pytest.py, and add it to the project, set it’s property content to yes.

```
def add(a,b):  
    return a + b
```

## 6. native code ##

```
#include <windows.h>
#if defined(_DEBUG)
#undef _DEBUG
#include "python.h"
#define _DEBUG
#else
#include "python.h"
#endif

void test()
{
	PyObject *pName, *pModule, *pDict, *pFunc, *pArgs, *pRetVal; 

    Py_Initialize();  
    if ( !Py_IsInitialized() ){  
        return;  
    }  
    pName = PyString_FromString("pytest");  
    pModule = PyImport_Import(pName);  
    if ( !pModule ){  
		OutputDebugString(L"can't find pytest.py\n");
        return;  
    }  
    pDict = PyModule_GetDict(pModule);  
    if ( !pDict ){  
        return;  
    }  
    pFunc = PyDict_GetItemString(pDict, "add");  
    if ( !pFunc || !PyCallable_Check(pFunc) ) {  
        OutputDebugString(L"can't find function [add]\n");  
		return;  
    }  
    pArgs = PyTuple_New(2);  
    PyTuple_SetItem(pArgs, 0, Py_BuildValue("l",3));   
    PyTuple_SetItem(pArgs, 1, Py_BuildValue("l",4));  
    pRetVal = PyObject_CallObject(pFunc, pArgs);  

	char ResultBuf[512];
	wchar_t ResultBufW[512];
	sprintf_s(ResultBuf,512,"function return value : %ld\r\n", PyInt_AsLong(pRetVal));
	MultiByteToWideChar( CP_ACP, MB_ERR_INVALID_CHARS, ResultBuf, -1,ResultBufW, 512 );

    OutputDebugString(ResultBufW);  
    Py_DECREF(pName);  
    Py_DECREF(pArgs);  
    Py_DECREF(pModule);  
    Py_DECREF(pRetVal);  

    Py_Finalize();  
    return;  
}

```

# source code and example can be downloaded from download page #