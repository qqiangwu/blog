In this post, I will briefly talk about chrome extensions which is really really awesome. Yes, this is just a reminder for myself.

Generally we can view chrome as an os, extensions as apps in the os. They can run in the background or add functionalities to pages we are working with.

A typical extension can be broken down into several parts, describled by a manifest.json file.

# Architecture
### The manifest file
```
{
    "name": "Powwow - Share your comments freely",
    "description": "Comment wherever you want",
    "manifest_version": 2,
    "version": "0.0.1",
    "background": {
        "page": "page/main.html"
    },
    "browser_action": {
        "default_icon": "img/icon.png",
        "default_popup": "page/popup.html",
        "default_title": "Powwow - Share your comments freely. Have fun!"
    },
    "content_scripts": [
        {
            "css": [
                "content_script/content.css"
            ],
            "js": [
                "lib/jquery.min.js",
                "lib/jquery.fittext.js",
                "lib/jquery.lettering.js",
                "lib/jquery.textillate.js",
                "content_script/content.js" 
            ],
            "matches": [ "http://*/*", "https://*/*" ],
            "run_at": "document_end"
        }
    ],
    "icons": {
        "16": "img/icon-16.png",
        "48": "img/icon-48.png",
        "128": "img/icon-128.png"
    },
    "permissions": [
        "storage"
    ],
    "web_accessible_resources": [
        "img/icon.png",
        "lib/jquery.min.map"
    ]
}
```

### Background page or background script
You can think a background page as a service to all pages, like Windows Service or Daemon Process.

There are two kinds of background pages: persistent background pages, and event pages. Unless you want your extension to be opened all the time, use event pages.

```
"background": {
    "page": "page/main.html"
}

"background": {
    "scripts": [
      "chrome_ex_oauthsimple.js",
      "chrome_ex_oauth.js",
      "background.js"
    ]
}

//! event pages
"background": {
    "scripts": ["eventPage.js"],
    "persistent": false
  }
```

#### Event pages
Loaded only when needed to save power and release system resources.

I won't talk about it until I really use it.

### UI or view
Basically, there are three kinds of UI:

1. Browser action: icon in the toolbar, typically used with a popup.
2. Page action: icon in the address bar.
3. Context menu: add items to the context menu.

When the extension concerns most of pages, use the brower action.

eg.

```
"browser_action": {
        "default_icon": "img/icon.png",
        "default_popup": "page/popup.html",
        "default_title": "Powwow - Share your comments freely. Have fun!"
    }
```

We can use `chrome.browerAction` to customize both the icon and the click handler. 

#### Popup
By default, we you click the icon in the toolbar, a popup will show.

#### Options page
A new way to set preferrences.

```
"options_page": "options.html"
```

#### View
All pages(the popup page, the option page, etc) used by a extension use `chrome.extension.getBackgroundPage()` to use service of the extension.

Since a view can either be opened or closed, the background script can only broadcast requests to interact with a view.

```JS
//! in a view
chrome.runtime.onMessage.addListener(function(message){
    if (message.cmd == "updateView") {
        view.updateView();
    }
});

//! in the background
chrome.runtime.sendMessage({ cmd: "updateView" });

// chrome.extension.getBackgroundPage()
// chrome.extension.getViews()
```

We can use `tab.create` to display views.

### Content scripts
We can inject scripts to the page we are browsing. Content scripts cannot interact with a extension directly, but indirectly. We use message passing to achieve this.

```JS
chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
  console.log(response.farewell);
});

chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {
    console.log(sender.tab ?
                "from a content script:" + sender.tab.url :
                "from the extension");
    if (request.greeting == "hello")
      sendResponse({farewell: "goodbye"});
  });
```

### Message Passing
We use `chrome.runtime.onMessage` and `chrome.runtime.sendMessage` to pass messages between difference components such as content scripts, views, and the background page.

`chrome.tab.*Message` are built on top of `chrome.runtime.*Message` and applied on to a single tab.

`chrome.extension.*Request` are deprecated.

# Chrome extension API
Besides html APIs, chrome offers a set of APIs used in extensions i.e. `chrome.*`. If we want to use an API, we must first get the permission.