# Integrate Trezor Connect with a web extension

[Example of a web extension project for both Google Chrome & Firefox](https://github.com/trezor/connect-explorer/tree/webextensions)

Basic implementation is same for both Google Chrome & Firefox. However, few additional steps are needed to make extension work with [WebUSB](https://wicg.github.io/webusb/) in Google Chrome.


1. Configure your manifest file ([Google Chrome](https://github.com/trezor/connect-explorer/blob/webextensions/manifest-chrome.json), [Firefox](https://github.com/trezor/connect-explorer/blob/webextensions/manifest-firefox.json))
    * Because Trezor Connect is served from the `https://connect.trezor.io/5` domain you must grant permissions to `://connect.trezor.io/5/*` URL in your manifest file.

        ```JSON
        {
            "permissions": [
                "*://connect.trezor.io/5/*"
            ]
        }
        ```
    * Include Trezor Connect as one of your background scripts

        If you're using a bundler only include final `index.js` file
        ```JSON
        {
            "background": {
                "scripts": [
                    "index.js"
                ]
            }
        }
        ```

        If you're not using a bundler you have to include Trezor Connect manually
        ```JSON
        {
            "background": {
                "scripts": [
                    "[myExtensionIndexFile]index.js",
                    "[pathToTrezorConnect]/index.js"
                ]
            }
        }
        ```

    * Include `trezor-content-script.js` in a `"content-scripts"`

        Trezor Connect may present a popup tab for certain actions. Since your code & Connect is running in a background script you need to allow communication between popup tab and background script explicitely using this Javascript [file](https://github.com/trezor/connect/blob/v5/src/js/webextension/trezor-content-script.js).

        ```JSON
        {
            "content_scripts": [
                {
                "matches": [
                    "*://connect.trezor.io/5/popup.html"
                ],
                "js": ["trezor-content-script.js"]
                }
            ],
        }
        ```
        Snippet above is basically saying _"Inject `trezor-content-script.js` into `connect.trezor.io/5/popup.html`"_.




2. Now you're able to use Trezor Connect in your code

    You can access `TrezorConnect` as a global variable if you included Trezor Connect in your project manually
    ```javascript
    function onClick() {
        TrezorConnect.getAddress({
            path: "m/49'/0'/0'/0/0"
        }).then(response => {
            const message = response.success ? `BTC Address: ${ response.payload.address }` : `Error: ${ response.payload.error }`;
            chrome.notifications.create(new Date().getTime().toString(), {
                type: 'basic',
                iconUrl: 'icons/48.png',
                title: 'TrezorConnect',
                message
            });
        }).catch(error => {
            console.error("TrezorConnectError", error)
        });
    }
    chrome.browserAction.onClicked.addListener(onClick);
    ```

    If you're using a package manager you will probably want to import Trezor Connect into your code using an `import` statement
    ```javascript
    import TrezorConnect from 'trezor-connect'; // Import Trezor Connect

    function onClick() {
        TrezorConnect.getAddress({
            path: "m/49'/0'/0'/0/0"
        }).then(response => {
            const message = response.success ? `BTC Address: ${ response.payload.address }` : `Error: ${ response.payload.error }`;
            chrome.notifications.create(new Date().getTime().toString(), {
                type: 'basic',
                iconUrl: 'icons/48.png',
                title: 'TrezorConnect',
                message
            });
        }).catch(error => {
            console.error("TrezorConnectError", error)
        });
    }
    chrome.browserAction.onClicked.addListener(onClick);
    ```

This is all that must be done in order to make Trezor Connect work with a web extension on Firefox.
However, if you're creating a Google Chrome extension you must complete one additional step.


## Google Chrome

Chrome extension requires a special `trezor-usb-permissions.html` file served from the root of extension - you can get the file [here](https://github.com/trezor/connect/blob/v5/src/js/webextension/trezor-usb-permissions.html).
This page is displayed in case a user is using Trezor without Trezor Bridge installed.

Lastly, you have to place [this](https://github.com/trezor/connect/blob/v5/src/js/webextension/trezor-usb-permissions.js) Javascript file into your `vendor/` folder.
