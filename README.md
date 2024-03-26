# Fixing the "Module 'firebase_core' not found" Error in iOS Build Process

When developing iOS applications with Flutter, integrating Firebase services is a common requirement. However, developers may encounter a frustrating error during the build process: "Module 'firebase_core' not found." This error indicates that the iOS build system cannot locate the Firebase Core module, which is essential for Firebase services to function correctly in your app. This tutorial will guide you through the process of fixing this error, both from the command line interface (CLI) and within a continuous integration/continuous deployment (CI/CD) workflow.

## Reproducing the Error

To understand how to fix the error, it's crucial to first reproduce it. Here's a step-by-step guide to encountering the "Module 'firebase_core' not found" error:

1. **Create a New Flutter Project**: Start by creating a new Flutter project using the command `flutter create my_app`.

2. **Add Firebase to Your Flutter Project**: Follow the Firebase documentation to add Firebase to your Flutter project. This typically involves adding the `firebase_core` package to your `pubspec.yaml` file and configuring your project with Firebase.

3. **Integrate Firebase Services**: Add other Firebase services (e.g., Firebase Authentication, Firestore) to your project by adding their corresponding Flutter packages to your `pubspec.yaml` file.

4. **Run Your App**: Attempt to run your app on an iOS simulator or device using the command `flutter run`. If everything is set up correctly, your app should build and run without issues. However, if there's a problem with the Firebase integration, you'll encounter the "Module 'firebase_core' not found" error.
5.  This also happens when running `flutter build ios` on a headleass ci/cd self hosted macos server for the first time

## Error Log

Here's an example of the error log you might see in your terminal or CI/CD logs:

  ```
Integrating client project
Pod installation complete! There is 1 dependency from the Podfile and 9 total pods installed.
Changing current working directory to: /Users/seth/actions-runner/_work/bookurtreat_mobile/bookurtreat_mobile
Warning: Building for device with codesigning disabled. You will have to manually codesign before deploying to device.
Building com.bookurtreat.bookurtreatMobile for device (ios)...
Running pod install...                                           2,297ms
Running Xcode build...                                          
Xcode build done.                                           70.6s
Failed to build iOS app
Parse Issue (Xcode): Module 'firebase_core' not found

```

## Fix
1.  **Confirm that `<flutter_project> ios/Runner.xcworkspace/Podfile` exists, if it doesn't exist, create one.
2.  **Add this to the Podfile:

   ```
def flutter_root
  generated_xcode_build_settings_path = File.expand_path(File.join('..', 'Flutter', 'Generated.xcconfig'), __FILE__)
  unless File.exist?(generated_xcode_build_settings_path)
    raise "#{generated_xcode_build_settings_path} must exist. If you're running pod install manually, make sure flutter pub get is executed first"
  end
  File.foreach(generated_xcode_build_settings_path) do |line|
    matches = line.match(/FLUTTER_ROOT\=(.*)/)
    return matches[1].strip if matches
  end
  raise "FLUTTER_ROOT not found in #{generated_xcode_build_settings_path}. Try deleting Generated.xcconfig, then run flutter pub get"
end
require File.expand_path(File.join('packages', 'flutter_tools', 'bin', 'podhelper'), flutter_root)
flutter_ios_podfile_setup
target 'Runner' do
  use_frameworks!
  use_modular_headers!
  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
  target 'RunnerTests' do
    inherit! :search_paths
  end
end
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
  end
end
```
3.  **From the ios folder run `flutter clean` followed by `flutter pub get` followed by `pod install`.
4.  Thats it. this works for building ios on a ci/cd server. tested on macos montrey 12.6 with flutter 3.19.4, firebase 10.x


