# Calling of Win32 API functions #
jN provides very powerful API working with Notepad++ and its views. But some times it is necessary to use Win32 API directly to get more inside or outside of Notepad++.



## The simplest call ##
We begin with the simplest call. It is [MessageBoxW](http://msdn.microsoft.com/en-us/library/windows/desktop/ms645505.aspx) defined in user32.dll

```
function MessageBox(caption, text){
  var lib = System.loadLibrary("user32.dll");
  var paramBuf = lib.alloc( 4 * 4);

  lib.writeDWord(paramBuf, 0,  0);
  lib.writeDWord(paramBuf, 4,  text);
  lib.writeDWord(paramBuf, 8,  caption);
  lib.writeDWord(paramBuf, 12, 0);

  lib.call("MessageBoxW", paramBuf);
}
```
Now we can call our native MessageBox as follows
```
MessageBox("Title of Box", "My important message");
```
What happened? First of all we load user32.dll into the address space of Notepad++. Then we allocate buffer for all parameters of function to be called. In our case MessageBox has 4 "Int32" parameters. We put parameter values into buffer. At the end we use call method of library instance to call MessageBoxW. It is unicode version of MessageBox. Call method retrieves address of MessageBoxW from user32.dll, places paramBuf on stack and then calls addressed function.

## Using of callback functions ##
Some of Win32 API functions use callback function mechanism to provide informations to us. I think it makes some algorithms more performant and reduces memory consumption. Our next candidate is [EnumWindows](http://msdn.microsoft.com/en-us/library/windows/desktop/ms633497.aspx). This function enumerates all top-level windows and uses a callback function to provide HWND of next window to calling program.
```
function EnumWindows(){
  var EnumWindowsProcCfg = {
    handles:[],
    cmd:function(buf){
           this.handles.push(buf);
           return true; // continue enumeration
	}
  };
  
  var EnumWindowsProc = System.registerCallback(EnumWindowsProcCfg, 2 * 4);
  
  var lib = System.loadLibrary("user32.dll");
  var paramBuf = lib.alloc(2 * 4);
  
  lib.writeDWord(paramBuf, 0, EnumWindowsProc);
  lib.writeDWord(paramBuf, 4, 0);
  
  lib.call("EnumWindows", paramBuf);
 
  return EnumWindowsProcCfg.handles;
};

alert(EnumWindows().length);

```

It is not possible to get an address of an JavaScript function and use it like Win32 function. Hence it was necessary to create native wrapper function that is able to call specified JavaScript function. To make this object oriented way we use an Object with required member function named _cmd_ instead of naked function. System.registerCallback function takes our _Cfg_ object and number of bytes necessary for parameters of callback and returns us a wraper object we can use like a pointer to a native callback function.

## Wrapper (Library.js) ##
Some time ago I begun a little javascript [wrapper](http://code.google.com/p/jn-npp-plugin/source/browse/trunk/deploy/%5BNotepad%2B%2B%20Directory%5D/plugins/jN/lib/Library.js) around jN's Win32 API. It offers easy way to declare Win32 functions and collect them in own libraries (e.g. [User32.dll.js](http://code.google.com/p/jn-npp-plugin/source/browse/trunk/deploy/%5BNotepad%2B%2B+Directory%5D/plugins/jN/lib/User32.dll.js)).

First of all we create new file for a dll where we define new global variable with a name of dll
```
require("lib/library.js"); // make library wrapper available

User32 = new Library("User32.dll"); // either a path to dll or it's name in case of system dlls
```
Now we can declare functions which that dll exports. For this purpose I go to documentation page of [function](http://msdn.microsoft.com/en-us/library/windows/desktop/ms645505%28v=vs.85%29.aspx) due to know count and types of parameters. Take care about unicode and ANSI names of a function (e.g. MessageBoxW and MessageBoxA). Prefer unicode one.
```
User32.Define("MessageBoxW", "HWND", "LPCTSTR", "LPCTSTR", "UINT");
```
After this is done User32 object has definition of MessageBoxW method expecting 4 parameters. Following code shows how to use it
```
User32.MessageBoxW(0, "My Message", "My Title", 0);
```
## Some thoughts about memory usage and GC ##
TODO