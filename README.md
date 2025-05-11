Announcement
------------
**Maintenance Notice**: As of May 2025, this repository (https://github.com/gstreamer101/gst-android-camera) will be actively maintained as the official fork of the original gst-android-camera project. We will provide regular updates, bug fixes, and support for modern Android versions. All future development and improvements for GStreamer Android camera integration will happen here.

If you're using the previous repository, we recommend updating your references to this one. Feel free to open issues or contribute via pull requests to help improve this project.


Example Android App for AHCSRC
==============================

Prerequisite
------------

 - Gstreamer SDK for Android (>=1.26.1)
 - Android Studio (>=2024.3.1)
 - Android NDK (>=r25c)
 - Gradle (>=8.11.1)

Build and Installation with terminal
----------------------

- Download GStreamer
Go to: https://gstreamer.freedesktop.org/download/
Download the Android Universal 1.26.1 tarball, and extract it to a desired location.

- Set GSTREAMER_ROOT_ANDROID evironment varible
```
  $ export GSTREAMER_ROOT_ANDROID=/path/to/gstreamer/sdk
```

 - Create `local.properties` and add `sdk.dir` properties

 - Build with gradle

```
  $ gradle build
```

 - Install with gradle

```
  $ gradle installDebug
```

Build and Installation with Android Studio
----------------------

- Set GSTREAMER_ROOT_ANDROID
  There are two ways to tell Gradle where the Android GStreamer is located

  * Set GSTREAMER_ROOT_ANDROID using Android Run Configurations
    * Run > Edit Configurations > + > Gradle
    * Environment variables
    * add : GSTREAMER_ROOT_ANDROID=/your/path

  * Set gstAndroidRoot using gradle.properties
  Open gradle.properties in your project and add:
    ```
    gstAndroidRoot=/your_gstreamer_path/gstreamer/1.26.1
    ```

- Sync Gradle

- Click the "Run 'app'" button at the top of Android Studio to build and install your app simultaneously.

Screenshots
----------
![screenshot](screenshots/screenshot.png)

Restrictions and Known bugs
---------------------------

 - ahcsrc element is a child element of GstPushSrc so it cannot be compatible
   with camerabin2 as it is.

 - No properties to set camera options like resolution, video formats.
   Each option can be set by caps filter.
   The options of real camera may vary on different android devices, so
   supported options should be analyzed on android application side.
