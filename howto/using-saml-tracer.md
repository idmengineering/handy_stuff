# HOW TO: Using SAML-Tracer to Capture SAML Event(s) for Debugging Purposes

> N.B. In our work, it'll frequently become necessary to ask customers (or our client's customers) to capture a SAML login or logout event, in order to assist our team in debugging the SAML workflow. This guide is therefore intended to serve as a walk-through for a potentially non-technical audience, to enable that user to install, use, and collect the log from the popular browser extension [SAML-Tracer](). 

SAML is most frequently leveraged in a browser-based flow. What this means is that your web browser (such as Google Chrome, Mozilla Firefox, or Microsoft Edge) will do all of the heavy lifting of exchanging data during the login or logout process. As such, it's often necessary that the person attempting a login (or logout) will need to "capture" that SAML data, as it's *their* browser that mediates the communication from an application (the "service provider", or SP) to the identity provider (or IdP) with which the user authenticates. This document will guide you step-by-step through installing and using the SAML-tracer browser add-on to capture a SAML trace.

## Step #1: Installing SAML-tracer

The SAML-Tracer add-on is available for free for Mozilla Firefox and Google Chrome from their respective web stores:

- Firefox: https://addons.mozilla.org/en-US/firefox/addon/saml-tracer/
- Chrome: https://chrome.google.com/webstore/detail/saml-tracer/mpdajninpobndbfcldcmbpnnbhibjmch?hl=en

You can also leverage the Chrome version of the plugin in Chromium-based browsers, such as:

- Microsoft Edge
- Vivaldi
- Brave

After you successfully install the extension, and to ensure that you have no active session, you should [clear your browser's cache](https://www.pcmag.com/how-to/how-to-clear-your-cache-on-any-browser) and restart the browser. 

💥<strong>PRO TIP</strong>💥 You can quickly access the menu for clearing your cache with the <kbd>ctrl + shift + delete</kbd> keyboard shortcut.

## Step #2: Capturing a SAML Login or Logout

After installing the extension in your browser one simply needs to activate the extension in order for it to capture traffic. To do this, either look for the SAML-tracer icon in your browser:

<p align="center">
    <img src="https://idmengineering.com/images/KE3dETB.png" alt="Screenshot of browser toolbar with SAML-tracer icon visible and highlighted by red arrowing pointing at it.">
    </img>
</p>
<p align="center">Figure 1. Mozilla Firefox toolbar with SAML-Tracer icon.</p>

In many browsers you'll need to "pin" the icon to be visible by default, e.g. for Mozilla Firefox, you may find the SAML-Tracer icon hiding under the dropdown that appears when clicking on the two angle brackets to the right side of the image. Also, in a stock Google Chrome installation, for example:

<p align="center">
    <img src="https://idmengineering.com/images/3zVidkn.png" alt="Screenshot of Chrome browser toolbar with SAML-tracer icon visible within extensions dropdown.">
    </img>
</p>
<p align="center">Figure 2. Google Chrome toolbar with SAML-Tracer icon hidden in the "puzzle piece" menu.</p>

You can see that the SAML-Tracer extension gets placed in the "Extensions" menu (denoted with a small puzzle piece). You can optionally pin the SAML-Tracer icon to the main toolbar with the "pin" button within that menu.

Once the extension is activated, you'll see a small pop-up window like the following:

<p align="center">
    <img src="https://idmengineering.com/images/a5VaPQy.png" alt="A screenshot of the initial window that opens when activating SAML-Tracer.">
    </img>
</p>
<p align="center">Figure 3. An empty SAML-Tracer window.</p>

You'll at this point potentially notice some lines appearing within the top-part of window. Ignore this for now. Go back to your browser, and log into the application you're trying to capture debug information for.

At this point, you'll see **many** more lines appear as your are performing your login. These lines represent all of the various URLs that are being loaded or to which data is being sent. After you login, you should notice that some of the lines will have an orange <img src="https://idmengineering.com/images/saml.png" alt="SAML logo."> logo to the right, for example:

<p align="center">
    <img src="https://idmengineering.com/images/2F8mGL9.png" alt="A screenshot of the SAML-Tracer extension showing captured SAML.">
    </img>
</p>
<p align="center">Figure 4. A screenshot of the SAML-Tracer extension showing captured SAML.</p>

If, when you click on one of the lines with this logo, and you also click on the "SAML" tab in the lower panel, you see some XML code as shown within the screenshot, then congratulations! You've captured SAML!

## Step #3: Sharing the Captured SAML

In order to share the capture you've just taken, click on the "export" button at the top of the SAML tracer window (next to "Colorize"). The window will darken, and a selection box will appear with some export options:

<p align="center">
    <img src="https://idmengineering.com/images/PqgiGYS.png" alt="A screenshot of the SAML-Tracer extension showing the options available for exporting captures.">
    </img>
</p>
<p align="center">Figure 5. The SAML-Tracer export options screen.</p>

**WARNING:** Make sure you either keep the "mask values" option checked, or select "Remove Values". If you chose "None" in this dialog, you will capture any secret parameters that were exchanged in the process, namely plaintext passwords. You do *NOT* need to share a capture with a plaintext password. Removing, or masking the value with a series of ****** are the only advisable options here.

When you click "export", a JSON file will be saved to the location of your choice. This is the file that you should share with the engineer that's requesting the SAML trace.

If requested from the IDM Engineer, we'll share with you a URL where you can securely upload the file, and it will be accessible only to the engineering team at IDME. Emailing this file is generally safe, though for added safety we recommend sharing the file through some means other than email.