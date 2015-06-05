# Working with menu #
jN allows us to create submenus of main menu of notepad++ at run time.
For better user experience we can enable and disable them depending on some states, if functionality not available at the moment.



## Video ##
<a href='http://www.youtube.com/watch?feature=player_embedded&v=3r3Bq_ceEQA' target='_blank'><img src='http://img.youtube.com/vi/3r3Bq_ceEQA/0.jpg' width='780' height=400 /></a>

## Creating a submenu ##
```
var cfg = { text: "My Menu" };
var myMenu = Editor.addMenu(cfg);
```

## Creating a menu entry ##
```
var myCmd = myMenu.addItem({
  // your own properties
  count: 0,
  // necessary properties
  text: "My command",
  cmd: function(m){
    this.count++; // access to your own properties
    alert(this.count);

    // Use myCmd variable or m to access
    // to menu entry object
    if (this.count == 3)
       m.disabled = true;
  }
});
```

In example above we add new menu entry "My command" with action code showing count of clicks we have done. We disable this menu entry if count of clicks exceeds 3.

![http://jn-npp-plugin.googlecode.com/svn/wiki/MenuItem.png](http://jn-npp-plugin.googlecode.com/svn/wiki/MenuItem.png)

## Creating separator ##
```
myMenu.addSeparator();
myMenu.addItem({text:"After separator"});
```

![http://jn-npp-plugin.googlecode.com/svn/wiki/MenuSeparator.png](http://jn-npp-plugin.googlecode.com/svn/wiki/MenuSeparator.png)

## Combine menu entry with shortcut ##
To improve user experience we can add shortcuts to our menu entries. It is possible to implement different action code for menu item and shortcut, but we combine them together. We do that by using the same configuration object _actionCfg_ by adding menu item and shortcut. The action code _cmd_ will be called during both shortcut and menu item click handling.

```
var actionCfg = {
  text: "Shortcut item\tCtrl+Shift+U", // relevant only for menu item
  cmd: function(m){ // m is valid only during menu item click
    alert(this.text + " handler");
  },
  
  // only shortcut relevant
  shift: true,
  ctrl: true,
  key:"u"
}
myMenu.addItem(actionCfg);
addHotKey(actionCfg);
```

![http://jn-npp-plugin.googlecode.com/svn/wiki/menushortcut.png](http://jn-npp-plugin.googlecode.com/svn/wiki/menushortcut.png)

We use **\t** to stretch place between menu item name and shortcut.

## Best way to disable whole submenu or single menu entry ##
Some times it is better way to not implement code enabling or disabling menu items as action on some state change. We want save CPU power and avaluate a collection of states only if menu item is going to get visible. For this purpose we extend our cfg object by adding _oninitpopup_ function and place there our evaluating code.
```
var toggleMenu = myMenu.addItem({text:""});
cfg.count= 0;
cfg.oninitpopup = function(){
  this.count++;
  toggleMenu.disabled = (this.count % 2) == 0;
  toggleMenu.text = "Update - "+this.count;
}
```

Every time we open "My Menu" _disabled_ property of new item will toggle and it's name shows updated count value.
![http://jn-npp-plugin.googlecode.com/svn/wiki/menuoninitpopup.png](http://jn-npp-plugin.googlecode.com/svn/wiki/menuoninitpopup.png)