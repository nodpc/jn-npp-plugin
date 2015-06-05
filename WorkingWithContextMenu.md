# Introduction #
Working with context menu means the way how javascript plugin developers can override default context menu of Notepad++ and improve so the user experience enormously.

# Details #
Following steps are necessary:
  1. Create own instance of context menu.
  1. Add code filling new context menu.
  1. Add code deciding if own or default context menu should be shown.
  1. Add code disabling some menu items dependend on current context.
  1. Add own context menu handler.

```
(function(){
	var ctxMenu = Editor.createContextMenu({oninitpopup:function(){
		// disable item if nothing selected
		selectionItem.disabled = currentView.selection == "";
	}});

	var selectionItem = ctxMenu.addItem({text:"Show selection", cmd:function(m){
		alert("Selection: " + currentView.selection);
	}})

	var showAlertItem = ctxMenu.addItem({text:"Show alert", cmd:function(m){
		alert("Always available menu item clicked");
	}})
		
	OnContextMenu = function(){
		// Show new context menu only for JavaScript files
		if (Editor.langs[currentView.lang] != "JS")
			return false;

		ctxMenu.show();
		
		return true;
	}


	GlobalListener.addListener({
		CONTEXTMENU: OnContextMenu
	})

})()
```

Use snippet above to realize context menus in your own plugins. Take a look into [API](http://jn-npp-plugin.googlecode.com/svn/wiki/API/api.xml) for more.

<a href='http://www.youtube.com/watch?feature=player_embedded&v=OmJVkP_qWw8' target='_blank'><img src='http://img.youtube.com/vi/OmJVkP_qWw8/0.jpg' width='820' height=500 /></a>