Some one asked me recently how to work with Scintilla directly and I wanted to write a simple example to explain that. But I understood it is difficult to do without knowing how jN and Scintilla are woking internally. For Scintilla there is very good documentation [page](http://www.scintilla.org/ScintillaDoc.html). This page should do this for jN.



# Prerequisites #
## Encoding ##
Keep in mind that scintilla works with UTF-8 encoded strings and jN uses MultiByte strings (constraint of used Scripting Engine). Hence every time you need to provide a string to Scintilla you have to convert it to UTF-8 first.

## Wrapper ##
I have begun a simple wrapper around scintilla (scintilla.js). Today it contains collection of constants and one method **Call**. The constructor expects a handle of scintilla's window. In our case it is **handle** property of **view**.

```
   var sci = new Scintilla(currentView.handle);
   var bytesOfDoc = sci.Call("SCI_GETLENGTH", 0, 0);
   alert(bytesOfDoc);
```

# Snippets #

## How to show autocompletition list box ##
The next example shows how to prepare entries of autocompletition list and show them.

```
(function(){

require("lib/scintilla.js")
require("lib/Kernel32.dll.js")

function toUtf8(str){
	// determine necessary count of bytes 
	var newlen = Kernel32.WideCharToMultiByte(65001, 0, str, str.length, 0, 0, 0, 0); // 65001 means UTF-8

	// prepare buffer
	var buf = Kernel32.NativeLibrary.alloc(newlen+1); // returns javascript string of newlen+1
	Kernel32.NativeLibrary.writeByte(buf, newlen, 0); // terminate encoded string with 0

	// encode string as UTF-8
	var reallen = Kernel32.WideCharToMultiByte(65001, 0, str, str.length, buf, newlen, 0, 0);
	return buf;
}

showAutoComplete = function(arr){ // Global scope
	var sci = new Scintilla(currentView.handle);

	var currentSep = String.fromCharCode(sci.Call("SCI_AUTOCGETSEPARATOR",0,0));
	var list = arr.join(currentSep);
	var listEncoded = toUtf8(list); // because Scintilla is working internally with UTF-8 
	
	sci.Call("SCI_AUTOCSHOW", 0, listEncoded);
}
})();

showAutoComplete(["hello","world"])
```

![http://jn-npp-plugin.googlecode.com/svn/wiki/scintilla_autocompletition_list_box.png](http://jn-npp-plugin.googlecode.com/svn/wiki/scintilla_autocompletition_list_box.png)

Be aware it is only an example without taking care about sort order, escaping of separator and so on. Read the [documentation](http://www.scintilla.org/ScintillaDoc.html#Autocompletion) first :)

## Folding of foldable region ##
Following code snippet shows how to fold a foldable region using javascript.
```
require("lib/Scintilla.js") // do it only once 
var sci = new Scintilla(currentView.handle);

// following call folds region beginning at line 17
sci.Call("SCI_TOGGLEFOLD", 17 - 1, 0);

```
Learn more from [Scintilla documentation](http://www.scintilla.org/ScintillaDoc.html#SCI_TOGGLEFOLD).

## Retrieve "Is Modified" flag of current file ##
Following code snippet shows how to get information if current document in first view is modified.
```
require("lib/Scintilla.js") // do it only once 
var sci = new Scintilla(firstView.handle);

var modified = sci.Call("SCI_GETMODIFY", 0, 0);

alert("modified "+(modified != 0))
```
Due to get collection of modified files one should iterate over all opened files of view, activate them temporarily and call "[SCI\_GETMODIFY](http://www.scintilla.org/ScintillaDoc.html#SCI_GETMODIFY)".