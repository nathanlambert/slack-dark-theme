# Slack Dark Theme

A theme that's a bit easier on the eyes.

# Installing into Slack

Find index.js file in your Slack's application directory. Will be something like:

* Windows: `%homepath%\AppData\Local\slack\resources\app.asar.unpacked\src\static\index.js`
* Mac: `/Applications/Slack.app/Contents/Resources/app.asar.unpacked/src/static/index.js`

At the very bottom, add

```js
// First make sure the wrapper app is loaded
document.addEventListener("DOMContentLoaded", function() {

   // Then get its webviews
   let webviews = document.querySelectorAll(".TeamView webview");

   // Fetch our CSS in parallel ahead of time
   const cssPath = 'https://cdn.rawgit.com/nathanlambert/slack-dark-theme/2ff56d50a7f4e5211a4eac5d2e9a7e1a585c52e8/custom.css';
   let cssPromise = fetch(cssPath).then(response => response.text());

   let customCustomCSS = `
   :root {
      /* Custom CSS here */
   }
   `

   // Insert a style tag into the wrapper view
   cssPromise.then(css => {
      let s = document.createElement('style');
      s.type = 'text/css';
      s.innerHTML = css + customCustomCSS;
      document.head.appendChild(s);
   });

   // Wait for each webview to load
   webviews.forEach(webview => {
      webview.addEventListener('ipc-message', message => {
         if (message.channel == 'didFinishLoading')
            // Finally add the CSS into the webview
            cssPromise.then(css => {
               let script = `
                     let s = document.createElement('style');
                     s.type = 'text/css';
                     s.id = 'slack-custom-css';
                     s.innerHTML = \`${css + customCustomCSS}\`;
                     document.head.appendChild(s);
                     `
               webview.executeJavaScript(script);
            })
      });
   });
});
```

* There is an area in the above code for custom CSS.
* Also notice that you can put any CSS URL you want, so feel free to fork this theme and put in your own URL.

That's it! Restart Slack and it'll take effect.

NB: You'll have to do this every time Slack updates.

# Forking & editing the theme

After forking, you'll want to edit your CSS. To do this, you can open Slack in your browser and run the following code in the console to preview the changes. You'll want to change the URL to the one that you're working with (keeping in mind that the most recent commit hash is in the URL since the GitHub CDN caches indefintiely for each URL):
```
$.ajax({
   url: 'https://cdn.rawgit.com/nathanlambert/slack-dark-theme/2ff56d50a7f4e5211a4eac5d2e9a7e1a585c52e8/custom.css',
   success: function(css) {
     $("<style></style>").appendTo('head').html(css);
   }
 });
 ```

# License

Apache 2.0
