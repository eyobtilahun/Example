With the release of Android 5.0, there are updated guidelines for modern Android UI design. These guidelines are called "material design". This page is dedicated to living in the "material world". 

<img src="https://i.imgur.com/bYWINtw.png" width="200" alt="screen 1" />
<img src="https://i.imgur.com/yMlrtTp.png" width="200" alt="screen 2" />
<img src="https://i.imgur.com/JZRCVkh.png" width="203" alt="screen 3" />

## What is Material Design?

What exactly is "Material Design"? This can be difficult to answer because of the broad scope of the updates. The concrete highlights of a material-designed app would include the following elements:

 1. [Vibrant Color Schemes](https://www.materialpalette.com)
 2. [[Updated Iconography|Material-Design-Primer#material-design-icons]]
 3. [[RecyclerView for Lists|Using-the-RecyclerView]] with items as [[CardViews|Using-the-CardView]]
 4. [[Leveraging the new App Toolbar|Using-the-App-ToolBar]] 
 5. [[Dynamic Scrolling Behavior with CoordinatorLayout|Handling-Scrolls-with-CoordinatorLayout]]
 6. [[Floating Action Buttons]] for important actions
 7. [[Ripple Animations for Touch Feedback|Ripple-Animation]]
 8. [[Shared Element Morph Transitions|Shared Element Activity Transition]] during navigation
 9. [[View Elevation and Shadows|Material-Design-Primer#elevation-and-shadows]] 
 10. [[Updated Dialog Themes|Material-Design-Primer#dialog-styles]]

This represents just a few of the highlights that represent the implementation of Material design.

## Design Guidelines

This new framework encompasses several changes to the interface of Android apps and Google strongly encourages the adoption of these new principles outlined below:

 * [Material Design Guidelines](http://www.google.com/design/spec/material-design/introduction.html) - Principles that guide material design
 * [Material Design Docs](https://developer.android.com/design/material/index.html) - Overview of material design
 * [Highly Simplified Material Design For Developers](http://www.materialdoc.com/components/) - Simplified Developer Guidlines
 * [Material Design for Developers](https://developer.android.com/training/material/index.html) - Developer guides for material 
 * [Material Design Examples](http://www.materialup.com/) - Blog posting beautiful material designs
 * [Material Palette](http://www.materialpalette.com/) - Easy material color selection
 * [Material Design Icons](http://materialdesignicons.com/) - Standard material icons from google and community

Using these materials, you can understand and begin adhering to these guidelines for your own apps.

## Material Design Setup

When switching a fresh material design theme, you can’t leave behind all older devices since the vast majority of current Android devices can't upgrade to lollipop. This is where the [Material Support Library](https://developer.android.com/training/material/compatibility.html#SupportLib) comes in to save the day. 

Google has released a handy [Material Design Checklist](http://android-developers.blogspot.ca/2014/10/material-design-on-android-checklist.html) to help ensure you've applied the necessary steps to adhere to the material look and feel.

### Include Support Library

By default, new generated apps are **automatically material design support enabled** within Android Studio. However, what about our old apps? How do we change them to support material design?  Take a look at this [[migration|Migrating-to-the-AppCompat-Library]] guide. 

### Design Support Library

As of June 2015, Google released the [[Design Support Library]] which helps simplify the use of a lot of material design components including a navigation drawer view, floating labels, floating action buttons, snackbars, and a new framework to tie motion and scroll events.   Follow the [[setup|Design-Support-Library#setup]] guide to take advantage of these new UI components.

### Change the Application Theme

Change your application theme to extend from a `Theme.AppCompat` theme in the `values/styles.xml` file:

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
  <item name="colorPrimary">@color/primary</item>
  <item name="colorPrimaryDark">@color/primary_dark</item>
  <item name="colorAccent">@color/accent</item>
</style>
``` 

These colors correspond to the following diagram:

<img src="https://developer.android.com/training/material/images/ThemeColors.png" width="300"/>

The available `Theme.Appcompat` which support material design include the following: `Theme.AppCompat`, `Theme.AppCompat.Light`, `Theme.AppCompat.Light.DarkActionBar`, `Theme.AppCompat.DeviceDefault`, `Theme.AppCompat.DeviceDefault.Light`, `Theme.AppCompat.DeviceDefault.Light.DarkActionBar`.

### Extend Activities from AppCompatActivity

`AppCompatActivity` is a subclass of `FragmentActivity`, so there should be no issues if you were using `FragmentActivity` from the older support library:

```java
public class HomeActivity extends AppCompatActivity {
    // Implementation
}
```

### Use Support ActionBar

See [[our ActionBar Guide|Extended-ActionBar-Guide#setting-up-actionbaractivity]] for more details on how to setup the compatibility ActionBar using `AppCompatActivity` including updating all of your `menu` files to use `app:showAsAction` instead of `android:showAsAction`.

Once you've switched to the `AppCompatActivity`, your code needs to access the ActionBar using `getSupportActionBar` instead of `getActionBar`:

```java
getSupportActionBar.show(); // RIGHT
getActionBar.hide(); // WRONG
```

In addition, styling the ActionBar needs to be done updated to use the [[material support styles|Extended-ActionBar-Guide#custom-actionbar-styles]] rather than the traditional ActionBar styles.

### Material Design Icons

With the Lollipop release comes an improved visual design style for icons used throughout modern Android apps. Google has provided a set of [material design icons](https://github.com/google/material-design-icons/) for developers to use within their apps.  You should [download the repo](https://github.com/google/material-design-icons/archive/master.zip) and grab the relevant files.  

<a href="http://google.github.io/material-design-icons/"><img width="500" src="https://i.imgur.com/VY4empy.png" alt="icons"></a>

If you are intending to use these icons for the [[Floating Action Buttons]], just make sure to pick one that is **24dp** according to the [[Google specs|Floating-Action-Buttons#dimensions]].

Be sure to check out the community-source [materialdesignicons](http://materialdesignicons.com/) icon site as well for an even larger selection of material icons.   If you wish to copy all the various density files to your project, you can use this [source code example](https://gist.github.com/rogerhu/6a56ad8a02159c266531) to help you copy the files through the command line.

Also, assuming your are using at least a [[Gradle|Getting-Started-with-Gradle]] version above 1.4.0 (see this [blog posting](http://android-developers.blogspot.com/2015/09/android-studio-14.html)), you can also import these icons as scalable vector graphics (.svg) files, which eliminates the need to import different image files for different resolutions.  You can right click on the `res/drawable` folder and click the new `Vector Asset` option to import these `.svg` files into a Vector Drawable XML, which can be used the same as any other [[drawable|Drawables]].

<img src="http://imgur.com/yYAI16t.png">

### Material Navigation Drawer

Take a look at the [Material Design Checklist](http://android-developers.blogspot.ca/2014/10/material-design-on-android-checklist.html) and you will notice that you need to change your existing drawer layout to  be over the toolbar and under the status bar. There’s a [handy StackOverflow post](http://stackoverflow.com/a/26440880/313399) explaining how to implement this.

<img src="https://i.imgur.com/hPOFJUf.gif" width="250" alt="drawer" />

Check out our [[Material Navigation Drawer|Fragment-Navigation-Drawer#usage]] guide for a step-by-step tutorial for setting up the updated material navigation drawer behavior. Note that to do this properly, you'll need to [[replace your ActionBar with a Toolbar|Using-the-App-ToolBar#using-toolbar-as-actionbar]].

## Material Views

### RecyclerView

With our material theme, there comes a much more flexible way to display lists of items called the  [[RecyclerView|Using-the-RecyclerView]], the spiritual successor to the `ListView`. 

![RecyclerView](https://i.imgur.com/bjKHkQFm.png)

In theory, we can still use either of these but Google recommends using the `RecyclerView` going forward. Check out the differences in our [[RecyclerView vs. ListView section|Using-the-RecyclerView#compared-to-listview]].

### CardView

Android 5.0 introduces a [[new widget called CardView|Using-the-CardView]] which is an easy way to decorate items in a list with a modern style.

<img src="https://developer.android.com/design/material/images/card_travel.png" width="200" />

`CardView` can be thought of as a FrameLayout with rounded corners and shadow based on its elevation. This is best used as part of a `ListView` or `RecyclerView` when displaying homogeneous content.

### Toolbar

With our material themes, there is now a spiritual successor to the `ActionBar` called the [[Toolbar|Using-the-App-ToolBar]]. 

![Toolbar](https://i.imgur.com/dGzDoDSm.png)

The toolbar operates as a replacement for the original `ActionBar` but is added directly to the layout XML file and can be placed and styled like any other view in the layout.

### Floating Action Buttons

[[Floating action buttons|Floating-Action-Buttons]] (or FAB) are a special "promoted action" within an activity or fragment that is moved from the `ActionBar` to a round icon floating above the UI in the bottom right corner.

<img src="http://i.imgur.com/BE7c4xy.gif" height="350" alt="FAB1" />&nbsp;
<img src="http://i.imgur.com/qij0f3K.gif" height="350" alt="FAB2" />

The floating action button should represent **the primary action** within a screen. More info and use cases of the FAB button can be found in Google’s official [design specs found here](http://www.google.com/design/spec/components/buttons.html#buttons-floating-action-button).

## Material Animations

Note that material animations are not compatible with Android versions prior to Lollipop API 21. See [this compatibility guide](https://developer.android.com/training/material/compatibility.html#CheckVersion) for more details. 

### Ripple Animations

Ripple provides an instantaneous visual confirmation at the point of contact when users interact with UI elements. Note that the ripples will only show up on devices running Lollipop, and will fall back to a static highlight on previous versions.

![Ripple](https://i.imgur.com/L9ZnabL.gif)

See our detailed [[ripple animations guide|Ripple-Animation]] for an overview of how to enable this effect on views and buttons.

### Shared Element Transitions

In many situations when navigating from a list of items to a detail view, there are elements common to both activities. With the new shared element transitions system, we can transition these shared elements from one activity or fragment to another creating a much more contiguous navigation effect.

![Shared Element Transition](https://i.imgur.com/1cjqsSA.gif)

See our detailed [[shared element transition guide|Shared-Element-Activity-Transition]] for an overview of how to enable these beautiful transitions. 

### Circular Reveal

[[Circular Reveal|Circular-Reveal-Animation]] is a new animation introduced in Android 5 that animates the view's clipping boundaries. Reveal animations provide users visual continuity when you show or hide a group of UI elements. 

![reveal](https://i.imgur.com/8jzWpX1.gif)

This animation is often times used in conjunction with a floating action button that will grow onto the screen using this transition.

### Scrolling Animations

[[Scroll effects|Handling-Scrolls-with-CoordinatorLayout]] can also be created to shrink or expand the Toolbar or Header to make room for the main content area as a user scrolls.

<img src="http://imgur.com/ah4l5oj.gif" alt="scrolling effects" width="300" />

## Material Styles

### Elevation and Shadows

Material design introduces elevation for UI elements. Elevation (controlling the Z-axis) determines the shadow cast by each view. You can set the elevation declaratively in your layouts, defined in `dp` for any view:

```xml
<ImageView …
    android:elevation="8dp" />
```

with this as the result:

![Example](https://i.imgur.com/QzaSPbI.png)

You can also set this from code using `getElevation()` or `setElevation()`. To customize the shadows or outlines of elevated views, check out the [official guide on shadows](https://developer.android.com/training/material/shadows-clipping.html#Shadows).

### Dynamic Color Palettes

Material Design encourages dynamic use of color, especially when you have rich images to work with. The [[new Palette support library|Dynamic-Color-using-Palettes]] lets you extract a small set of colors from an image to style your UI controls to match; creating an immersive experience. 

![Palette](https://i.imgur.com/nwtguAS.png)

The extracted swatches will include vibrant and muted tones as well as foreground text colors for optimal legibility. This allows us to use these dynamically selected colors to apply better color selections to our layouts.

### Dialog Styles

If you want your Dialogs to have a Material look and feel across all Android versions, you’ll want to use a library to achieve modern dialogs such as:

 * [material-dialogs](https://github.com/afollestad/material-dialogs)
 * [AlertDialogPro](https://github.com/fengdai/AlertDialogPro/)
 * [MaterialDialog](https://github.com/drakeet/MaterialDialog)

![Material Dialog](https://i.imgur.com/U4mr2BB.jpg)

## References

* <http://android-developers.blogspot.ca/2014/10/material-design-on-android-checklist.html>
* <http://www.google.com/design/spec/material-design/introduction.html>
* <http://code.hootsuite.com/tips-and-tricks-for-android-material-support-library/>
* <http://code.hootsuite.com/tips-and-tricks-for-android-material-support-library-2-electric-boogaloo/>
* <http://android-developers.blogspot.com/2015/06/more-material-design-with-topeka-for_16.html>