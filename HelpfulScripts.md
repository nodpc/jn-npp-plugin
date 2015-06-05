On this page I will collect short snippets I needed someday. Scintilla relevant snippets you will find [here](DirectCallsToScintilla.md).



# How to save current document with certain name/path #
Because of API of Notepad++ do not provide a possibility to save file with certain name, we can try to implement a workaround.

We use the fact that Save As dialog pop ups with activated file name text box. So we can paste our file name in this box and "press" enter. Drawback of this way is, you will see this dialog shortly and you have to care that file does not exist.
```
function SaveAs(newFileName){
    // go around the blocking FILE_SAVE
    System.setTimeout({millis:100,cmd:function(){
        var shell = new ActiveXObject("WScript.Shell");
        shell.SendKeys(newFileName);
        shell.SendKeys("{ENTER}");
    }});

    // blocks executing of script
    MenuCmds.FILE_SAVEAS();
}
```

More information about [WScript.Shell](http://msdn.microsoft.com/en-us/library/8c6yea83%28v=vs.84%29.aspx)

# How to use system "Browse for folder" dialog #
```
var shellApp = new ActiveXObject("shell.application");

var folder = shellApp.BrowseForFolder(0,"Select my folder",0, "c:\\");
if (folder != null)
	alert(folder.self.Path);
else
	alert("Canceled");

```
First parameter is a handle of a parent window. It is possible to use Editor.handle for this purpose. The second parameter is a text message shown at the top of dialog. Third parameter is a flag for a dialog. Fourth parameter is optional start node.

More information about [Shell.BrowseForFolder](http://msdn.microsoft.com/en-us/library/windows/desktop/bb774065%28v=vs.85%29.aspx)

# Read from file #
In following snippet we use [ADODB.Stream](http://msdn.microsoft.com/en-us/library/windows/desktop/ms677486%28v=vs.85%29.aspx) ActiveX for reading from file. ADODB.Stream allows to use certain [character set](http://msdn.microsoft.com/en-us/library/windows/desktop/ms681424%28v=vs.85%29.aspx) (UTF-8, Windows-1252, ..) for reading files.
```
function readFile(path, charset) {
	if (typeof(charset) == "undefined")
		charset = "Windows-1252";
		
	var stream = new ActiveXObject("ADODB.Stream");

	stream.Charset = charset;
	stream.Open();
	stream.LoadFromFile(path);

	var result = stream.ReadText();
	stream.close();

	return result;
}

// test code
try{
	alert(readFile("d:\\my-utf8-file.txt","UTF-8"));
	alert(readFile("d:\\my-ansi-file.txt"));
}catch(e){
	alert(e.message)
}
```

# Read registry keys using WMI #
The next code using WMIs [StdRegProv](http://msdn.microsoft.com/en-us/library/aa393664%28v=vs.85%29.aspx) is very tricky, because javascript does not support out parameters and the call we use returns value over it. It was a good message for me, that there is a [way](http://msdn.microsoft.com/en-us/library/aa389296%28v=vs.85%29.aspx) to do it.
```
function readRegSubKeys(keyNum, subKeyName){
	//Constructing InParameters Objects and Parsing OutParameters Objects
	
	var objReg=GetObject("winmgmts:{impersonationLevel=impersonate}!//./root/default:StdRegProv");
	
	// because of javascript does not support out parameters do all things other
	// InParameters
	var objInParam = objReg.Methods_("EnumKey").InParameters.SpawnInstance_();


	// Add the input parameters.
	objInParam.Properties_.Item("hDefKey")     =  keyNum;
	objInParam.Properties_.Item("sSubKeyName") =  subKeyName;

	// Execute the method and obtain the return status.
	// The OutParameters object in objOutParams
	// is created by the provider.
	var objOutParams = objReg.ExecMethod_("EnumKey", objInParam);
	
	// read output parameter. For StdRegProv it is sNames
	var sNames = objOutParams.Properties_.item("sNames").Value;
	
	// because of sNames is a SafeArray convert it to javascript array
	return  (new VBArray(sNames)).toArray();
}

try{

	var HKEY_CLASSES_ROOT  = 0x80000000;

	alert(readRegSubKeys(HKEY_CLASSES_ROOT, "MIME\\Database\\Charset").join(", "));
}catch(e){
	alert(e.message)
}
```