## Plugins and npm modules

As you build more complex apps, you'll likely run into functionality that is not implemented in the NativeScript modules. But no worries, as NativeScript lets you leverage [npm](https://www.npmjs.com/) (node package manager) to import npm modules into your apps. Alternately, you can install NativeScript plugins, which are simply npm modules that can access native code and use Android and iOS SDKs, if required. 

In this chapter, you'll install and use an external email validator module to verify the format of email addresses as they are entered on the registration screen. Then, you'll add a NativeScript plugin, [NativeScript social share](https://www.npmjs.com/package/nativescript-social-share), to let users share their grocery lists using their device's native sharing widget.

### Using npm modules

It would be nice to be able to make sure people are entering well-formatted email addresses into your app on the registration screen. You could write this functionality yourself, but validating email addresses is [surprisingly tricky](http://stackoverflow.com/questions/46155/validate-email-address-in-javascript), and it's a lot easier to use one of many npm modules that already provide this validation. For Groceries let's see how to add this [email-validator module](https://www.npmjs.com/package/email-validator) to test for valid addresses.

<h4 class="exercise-start">
    <b>Exercise</b>: Install the email validator module
</h4>

Return to your terminal and make sure that you are working in the root directory in your Groceries project folder, a.k.a. here:

<div class="no-copy-button"></div>

```
sample-Groceries <----------------
    ├── app
    │   └── ...
    ├── package.json
    └── platforms
        ├── android
        └── ios
```

From the root directory install the email-validator module:

```
npm install email-validator --save
```

<div class="exercise-end"></div>

The install process does a few things in the background. First, because you added the `--save` flag, npm records this dependency in your app's `package.json`. If you open your `package.json` you should see `"email-validator"` in your app's `"dependencies"` array.

``` JavaScript
"dependencies": {
  "email-validator": "^1.0.4"
}
```

The npm CLI also creates a `node_modules` folder in the root of your app. This folder contains the code for the email-validator module, which is a bit of validation logic in `node_modules/email_validator/index.js`. 

> **TIP**: By saving your app's npm dependencies in your `package.json` file, you can always regenerate your `node_modules` folder by running `npm install`. Because of this, it's a common practice to exclude the `node_modules` folder from source control. The Groceries app uses git for source control, and as such includes `node_modules/` in its `.gitignore`.

Now that you have the module installed let's look at how to use it.

<h4 class="exercise-start">
    <b>Exercise</b>: Use the email validator module
</h4>

Open `/app/shared/user/user.ts` and replace the existing contents of the file with the code below:

``` TypeScript
var validator = require("email-validator");

export class User {
  email: string;
  password: string;
  isValidEmail() {
    return validator.validate(this.email);
  }
}
```

> **NOTE**: The NativeScript framework's `require()` method is configured to look at the `"main"` value in an npm module's `package.json` file. In the case of this module, the `"main"` value is `"index.js"`. Therefore, when you run `require("email-validator")`, you're actually requiring the file at `node_modules/email_validator/index.js`. You could also type `require("email-validator/index")` to retrieve the same file.

To make use of this validator, open `app/pages/login/login.component.ts` and paste the following code at the beginning of the existing `submit()` function:

``` TypeScript
if (!this.user.isValidEmail()) {
  alert("Enter a valid email address.");
  return;
}
```

<div class="exercise-end"></div>

Explain this code.

TODO: Add images

In general npm modules greatly expand the number of things you're able to do in your NativeScript apps. Need date and time formatting? Use [moment](https://www.npmjs.com/package/moment). Need utility functions for objects and arrays? Use [lodash](https://www.npmjs.com/package/lodash) or [underscore](https://www.npmjs.com/package/underscore). This code reuse benefit gets even more powerful when you bring NativeScript plugins into the picture.

> **WARNING**: Not all npm modules work in NativeScript apps. Specifically, modules that depend on Node.js or browser APIs will not work, as those APIs do not exist in NativeScript. The NativeScript wiki contains a [list of some of the more popular npm modules that have been verified to work in NativeScript apps](https://github.com/NativeScript/NativeScript/wiki/supported-npm-modules).

### Using NativeScript plugins

NativeScript plugins are npm modules that have the added ability to run native code and use iOS and Android frameworks. Because NativeScript plugins are just npm modules, a lot of the techniques you learned in the previous section still apply. The one big difference is in the command you use to install plugins. Let's look at how it works by installing the NativeScript social share plugin.

<h4 class="exercise-start">
    <b>Exercise</b>: Install the social sharing plugin
</h4>

Return to your terminal, make sure you're still in the root of your app, and run the following command:

```
tns plugin add nativescript-social-share
```

<div class="exercise-end"></div>

The install process does the same thing that the `npm install` command does—including retrieving the module from npm, installing the module in `node_modules`, and saving the module as a dependency in your app's `package.json`—but the `tns plugin add` command additionally configures any native code that the plugin needs to use.

For example the [NativeScript push plugin](https://github.com/NativeScript/push-plugin) uses both iOS and Android SDKs, and the `tns plugin add` command takes care of installing those. The [NativeScript flashlight plugin](https://github.com/tjvantoll/nativescript-flashlight) needs permissions to use the camera on Android, and the `tns plugin add` command takes care of setting that up too.

Now that you've installed the social share plugin, let's look at how to use it.

<h4 class="exercise-start">
    <b>Exercise</b>: Use the social sharing plugin
</h4>

Open `app/pages/list/list.component.ts` and add the following line at the top of the file, which requires the social share module you just installed:

``` JavaScript
var socialShare = require("nativescript-social-share");
```

Next you have to build some UI that lets you share a grocery list. To do so, open `app/pages/list/list.html` and add the following code at the very top of the file:

``` XML
<ActionBar title="Groceries">
  <ActionItem text="Share" (tap)="share()" android.systemIcon="ic_menu_share_holo_dark" ios.systemIcon="9" ios.position="right"></ActionItem>
</ActionBar>
```

This code defines an [ActionBar]({{site.baseurl}}/ApiReference/ui/action-bar/ActionBar), which is a UI component that includes various menu items, or `<ActionItem>` components.

> **NOTE**: On iOS devices, `<ActionItem>`s are placed from left to right in sequence; you can override that (as the code above does) by providing an `ios.position` attribute.

Next, to add a bit of styling to this new `<ActionBar>`, add the following CSS to your `app/app.css` file:

``` CSS
ActionBar {
  background-color: black;
  color: white;
}
```

Finally, now that you’ve installed and required the plugin, and setup a UI to use it, your last step is implementing the `<ActionItem>`'s `tap` handler (`share()`). Open `app/pages/list/list.component.ts` again and add the following function to the `ListPage` class:


``` TypeScript
share() {
  var list = [];
  for (var i = 0, size = this.groceryList.length; i < size ; i++) {
    list.push(this.groceryList[i].name);
  }
  var listString = list.join(", ").trim();
  socialShare.shareText(listString);
}
```

<div class="exercise-end"></div>

This code takes the grocery data from the grocery list array, converts the data into a comma-separated string, and passes that string to the social share plugin’s `shareText()` method.

Now when you run the app, you'll see a new button at the top of the screen. When you tap it, the native iOS or Android sharing widget will show to let you post your groceries to your social networks, or send them via email, message, or any other method you prefer.

TODO: Images

Pretty cool, huh? The ability to use npm modules greatly expands the number of things you're able to do in a NativeScript app. Need to compose emails in your app? Try out the [NativeScript email plugin](https://www.npmjs.com/package/nativescript-email). Need to use the clipboard in your app? Try out the [NativeScript clipboard plugin](https://www.npmjs.com/package/nativescript-clipboard).

If you're looking for NativeScript plugins start by searching both the [Telerik NativeScript Plugins Marketplace](http://plugins.telerik.com/nativescript) and our [community-curated list of plugins on npm](http://plugins.nativescript.rocks). If you don't find the plugin you need, you can [request the plugin on our ideas portal](https://nativescript.ideas.aha.io/), or you can take a stab at [creating the plugin yourself](https://docs.nativescript.org/plugins).

Between NativeScript modules, npm modules, and NativeScript plugins, the NativeScript framework provides a lot of functionality you can use to build your next app. However, we've yet to talk about NativeScript's most powerful feature: the ability to directly access iOS and Android APIs in JavaScript. Let's look at how it works.