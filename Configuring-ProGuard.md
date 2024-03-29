ProGuard is a tool to help minify, obfuscate, and optimize your code.  It is not only especially useful for reducing the overall size of your Android application as well as removing unused classes and methods that contribute towards the intrinsic [64k method limit](http://developer.android.com/tools/building/multidex.html#avoid) of Android applications.  Because of the latter issue, ProGuard is often recommended to be used both in development and production especially for larger applications.  

ProGuard can be enabled by using the `minifyEnabled` option for any build type.  If you intend to use it for production, it is highly recommended you also enable it on your development.  Without fulling testing ProGuard on your development builds, you may encounter unexpected crashes or situations where the app does not function as expected.

```gradle
android {
   buildTypes {
      dev {
          minifyEnabled true  // enables ProGuard
          proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
}
```

The Android SDK comes with ProGuard included as well as default settings file, which are specified as [proguard-android.txt](https://android.googlesource.com/platform/sdk/+/master/files/proguard-android.txt).  It is important to include this file since it explicitly includes configuration settings such as explicitly stating that all View getter and setter methods should not be removed.   The `proguard-rules.pro` file is the file that you will use to configure.

### Caveats

Proguard can add a few minutes to your build cycle.  If you can avoid using ProGuard in development, you should try to do so.  Once you begin to include enough libraries that causes the 64K method limit to be reached, you either need to remove extraneous dependencies or need to consider following the instructions for supporting a higher limit by using the [multidex mode](http://developer.android.com/tools/building/multidex.html).  Multidex compilation also takes additional time and requires extra work to support pre-Lollipop Android versions, so the recommendation is often to use ProGuard before using Multidex.

#### Checking method limit

If you wish to check how close you are to the 64k limit, fork a copy of the [dex-method-counts](https://github.com/mihaip/dex-method-counts) library.  Inside this repo, type:

```bash
./gradlew assemble
```

Once this project compiles successfully, you can point this program to any of your `.apk` files (normally located in `app/build/outputs/apk`:

```bash
./dex-method-counts build/outputs/debug//myapp.apk # or .zip or .dex or directory
```

The program will generate a breakdown of the methods used across each library.

### Development

When using in development/debug testing, you may wish to turn on a few settings that may add to compile time as well as make it harder to troubleshoot.  For instance, to ensure that no code optimizations or obfuscation is done, the following options should be declared in your ProGuard config:

```java
# Workaround for ProGuard not recognizing dontobfuscate
# https://speakerdeck.com/chalup/proguard
-dontobfuscate
-dontoptimize
-optimizations !code/allocation/variable
```

### Third-Party Libraries

Before you start adding any ProGuard rules, you should also check whether any of the libraries you use already come packaged with a `consumer-proguard-rules.pro` file.  This file gets added to the ProGuard definition list automatically.  For instance, for the [LeakCanary](https://github.com/square/leakcanary) project, the definition listings are located [here](https://github.com/square/leakcanary/blob/master/leakcanary-android/consumer-proguard-rules.pro) and added whenever you use this library with ProGuard enabled.

In addition, you can either check the GitHub source to see if this file exists, or you can navigate through Android Studio by clicking on the `Project` tab and navigating to `app` -> `build` -> `intermediates` -> `exploded-aar` and looking for the `proguard.txt` definition. 

<img src="http://imgur.com/2ZY2aSG.png"/>

#### Common ProGuard configs

One limitation in ProGuard is that dynamically generated classes such as those used in Butterknife, Otto, or Dagger 2 need to be explicitly declared.  In many cases, compilation will not continue until these issues are resolved.  In other cases, if you do not specify it, ProGuard will often strip these methods from being used and your app will often crash at run-time.   

See [this link](https://github.com/krschultz/android-proguard-snippets/tree/master/libraries) for the standard set of library configs.  It's important to check the documentation of each third-party library to see the most updated settings.  You can often try to experiment too by incrementally adding the necessary lines to best understand the impact of each line configuration.

Here is an example of some of the ProGuard definitions for various popular libraries.  

#### ButterKnife

**Note**: The following ProGuard lines apply to ButterKnife v7.0.  If you are using an older version of ButterKnife, the Proguard rules may be slightly different.

```
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }

-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}

-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}
```

#### Retrofit

```
-dontwarn retrofit.**
-keep class retrofit.** { *; }
-keepattributes Signature
-keepattributes Exceptions
```

#### OkHttp

```
-keepattributes Signature
-keepattributes *Annotation*
-keep class com.squareup.okhttp.** { *; }
-keep interface com.squareup.okhttp.** { *; }
-dontwarn com.squareup.okhttp.**

# Okio
-keep class sun.misc.Unsafe { *; }
-dontwarn java.nio.file.*
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
```

#### Gson

```
-keep class sun.misc.Unsafe { *; }
-keep class com.google.gson.stream.** { *; }
```

#### Parcels

```
keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}

-keep class org.parceler.Parceler$$Parcels
```

### Troubleshooting

If you wish to confirm whether ProGuard is preserving certain annotations or classes, you can review the `.apk` package that gets created to check.  The first step is to download the [dex2jar](http://sourceforge.net/projects/dex2jar/files/) program and use it to decompile the Dalvik code (`.dex` file) to a Java archive file (`.jar` file).  

```bash
wget http://sourceforge.net/projects/dex2jar/files/dex2jar-2.0.zip/download -O dex2jar-2.0.zip
unzip dex2jar-2.0.zip
chmod u+x ~/projects/dex2jar-2.0/*.sh
./d2j-dex2jar.sh <.apk file>
```

You can also use `d2j-dex2jar.bat` if using a Windows machine:

```dos
d2j-dex2jar.bat <.apk file>
```

Running the `dex2jar` file directly on an APK file should convert it to a `.jar` file.  You can download [JD-GUI](http://jd.benow.ca/) and open this newly created file to review the Java class files.  The UI allows you to double-check whether certain annotations were removed and whether certain classes were kept in the final compilation.


### References

* <https://github.com/krschultz/android-proguard-snippets/tree/master/libraries/>
* <http://developer.android.com/tools/help/proguard.html>