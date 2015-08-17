# PermissionsDispatcher

[![Build Status](https://travis-ci.org/hotchemi/PermissionsDispatcher.svg)](https://travis-ci.org/hotchemi/PermissionsDispatcher)
[ ![Download](https://api.bintray.com/packages/hotchemi/maven/permissionsdispatcher/images/download.svg) ](https://bintray.com/hotchemi/maven/permissionsdispatcher/_latestVersion)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-PermissionsDispatcher-green.svg?style=flat)](https://android-arsenal.com/details/1/2316)


> **PermissionsDispatcher is based on the latest [official document](https://developer.android.com/preview/overview.html). Currently preview2 is the newest but it may change drastically in the near future. 1.0 will be out after M final release.**

PermissionsDispatcher provides simple annotation-based API to handle runtime permissions in Android M.

You can be released from the burden that writing a bunch of check statements whether a permission have been granted or not. 

## Motivation

[Runtime permissions](https://developer.android.com/preview/features/runtime-permissions.html) is so great for users but also the hell for developers. Actually I don't want to write such a complicated [code](https://github.com/googlesamples/android-RuntimePermissions/blob/master/Application/src/main/java/com/example/android/system/runtimepermissions/MainActivity.java) anymore.

## Usage

Here's a minimum example that you register `MainActivity` which requires `Manifest.permission.CAMERA`.

### 1. Attach annotations

There are only 3 annotations.

- `@RuntimePermissions`: [Must] Register an Activity or Fragment to handle permissions.
- `@NeedsPermission`: [Must] Register a method which the permission is needed.
- `@ShowsRationale`: [Option] Register a method which explains why the permission is needed. Actually an annotated method is called when `shouldShowRequestPermissionRationale` returns true.

> NOTE: Annotated methods must be package private or above.

```java
@RuntimePermissions
public class MainActivity extends Activity {

    @NeedsPermission(Manifest.permission.CAMERA)
    void showCamera() {
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance())
                .addToBackStack("camera")
                .commitAllowingStateLoss();
    }

    // this is option
    @ShowsRationale(Manifest.permission.CAMERA)
    void showRationaleForCamera() {
        Toast.makeText(this, R.string.permission_camera_rationale, Toast.LENGTH_SHORT).show();
    }
}
```

### 2. Delegate to generated class

Now you can use the class that generated by annotation processor.

In this case the name is `MainActivityPermissionsDispatcher`.

It has methods ends with `WithCheck` and `onRequestPermissionsResult`.

Only you have to do is delegating the work to them.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Button cameraButton = (Button) findViewById(R.id.button_camera);
    cameraButton.setOnClickListener(this);
    Button contactsButton = (Button) findViewById(R.id.button_contacts);
    contactsButton.setOnClickListener(this);
}

@Override
public void onClick(View v) {
    switch (v.getId()) {
        case R.id.button_camera:
            // NOTE: delegate the permission handling to generated method
            MainActivityPermissionsDispatcher.showCameraWithCheck(this);
            break;
        case R.id.button_contacts:
            // NOTE: delegate the permission handling to generated method
            MainActivityPermissionsDispatcher.showContactsWithCheck(this);
            break;
    }
}

@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    // NOTE: delegate the permission handling to generated method
    MainActivityPermissionsDispatcher.
            onRequestPermissionsResult(this, requestCode, permissions, grantResults);
}
```

If you want to know more detail, please check the [sample](https://github.com/hotchemi/PermissionsDispatcher/tree/master/permissionsdispatcher-sample) and [generated class](https://gist.github.com/hotchemi/5dc89c02399c2419fab7).

## Download

Add to your project `build.gradle` file:

```groovy
buildscript {
  dependencies {
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
  }
}

apply plugin: 'android-apt'

dependencies {
  compile 'com.github.hotchemi:permissionsdispatcher:0.9.0'
  apt 'com.github.hotchemi:permissionsdispatcher-processor:0.9.0'
}
```

## ProGuard

```
-dontwarn permissions.dispatcher.processor.**
-keep class permissions.dispatcher.** { *; }
-keep class **PermissionsDispatcher { *; }
-keepclasseswithmembernames class * {
    @permissions.dispatcher.* <methods>;
}
```

## License

```
Copyright 2015 Shintaro Katafuchi

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
