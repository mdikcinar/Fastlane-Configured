# Fastlane Configuration

This directory contains Fastlane configuration files for automating iOS and Android deployment processes.

## File Structure

```
fastlane/
├── Fastfile              # Main Fastlane configuration file that imports all lanes
├── configs.rb            # Environment variables and configurations
├── configs_example.rb    # Example configuration file (for reference)
├── helpers.rb            # Helper methods used across lanes
├── common_lanes.rb       # Common lanes used by both platforms
├── android_lanes.rb      # Android specific lanes
└── ios_lanes.rb          # iOS specific lanes
```

## Files Description

### Fastfile

Main configuration file that imports all other lane files. This file serves as an entry point for all Fastlane commands.

### configs.rb

Contains environment variables for:

- App Store Connect API credentials
- Google Play Store JSON key path
- Slack webhook URL
Note: This file should not be committed to version control.

### configs_example.rb

Example configuration file showing the required environment variables structure. Use this as a template to create your own `configs.rb`.

### helpers.rb

Contains helper methods used across different lanes:

- `get_flutter_version`: Reads version from pubspec.yaml
- `validate_build_options`: Validates Android/iOS build options
- `validate_ios_build_options`: Validates iOS specific build options
- `validate_app_store_credentials`: Validates App Store credentials

### common_lanes.rb

Contains lanes used by both platforms:

- `add_git_tag_method`: Adds git tag for releases
- `send_slack_message`: Sends deployment notifications to Slack

### android_lanes.rb

Android specific lanes:

- `read_version`: Reads version from pubspec.yaml
- `increment_build_number_lane`: Increments Play Store build number
- `increment_firebase_build_number_lane`: Increments Firebase build number
- `flutter_build`: Builds Android app
- `build_store`: Uploads to Play Store
- `upload_to_firebase`: Uploads to Firebase App Distribution
- `deploy_android`: Main deployment lane for Play Store
- `deploy_android_firebase`: Main deployment lane for Firebase

### ios_lanes.rb

iOS specific lanes:

- `connect_app_store`: Sets up App Store Connect API
- `read_and_increment_build_number`: Updates version and build numbers
- `build_app_lane`: Builds iOS app
- `upload_to_testflight_lane`: Uploads to TestFlight
- `upload_symbols_to_crashlytics_lane`: Uploads dSYM to Crashlytics
- `deploy_ios`: Main deployment lane for TestFlight

## Usage

### Project-Specific Configuration

Each app in the monorepo can have its own Fastfile and Appfile in their respective directories:

```
apps/
└── your_app/
    ├── android/
    │   └── fastlane/
    │       ├── Fastfile    # Android specific configuration
    │       └── Appfile     # Android credentials and app identifiers
    └── ios/
        └── fastlane/
            ├── Fastfile    # iOS specific configuration
            └── Appfile     # iOS credentials and app identifiers
```

#### Example iOS Fastfile

```ruby
# apps/better_iptv/ios/fastlane/Fastfile

default_platform(:ios)

# Import the shared Fastfile containing the deploy lane logic
import '../../../../bin/fastlane/Fastfile'

platform :ios do
  desc "Deploy production application to TestFlight"
  lane :prod do
    deploy_ios(
      scheme: "production",
      app_name: "Example APP",
      slack_message: "Example APP iOS: Production app successfully released!",
    )
  end
end
```

#### Example iOS Appfile

```ruby
# apps/example_app/ios/fastlane/Appfile

# If not using common configs.rb, configure app-specific credentials here
ENV['APP_STORE_KEY_ID'] = "Example123"
ENV['APP_STORE_ISSUER_ID'] = "1234567890"
ENV['APP_STORE_KEY_FILENAME'] = "AuthKey_Example123.p8"
ENV['SLACK_URL'] = "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"

# You can use environment variables or provide static values
for_platform :ios do
  for_lane :prod do
    app_identifier 'com.example.exampleapp'
  end
  for_lane :dev do
    app_identifier 'com.example.exampleapp.dev'
  end
end

```

#### Example Android Fastfile

```ruby
# apps/better_iptv/android/fastlane/Fastfile

default_platform(:android)

# Import the shared Fastfile containing the deploy lane logic
import '../../../../bin/fastlane/Fastfile'

platform :android do
  desc "Deploy production application to Play Store"
  lane :prod do
    deploy_android(
      track: "internal",
      app_name: "Example APP",
      app_identifier: "com.example.exampleapp",
      flavor: "production",
      target: "lib/main_production.dart",
      slack_message: "Example APP Android: Production app successfully released!"
    )
  end

  desc "Deploy development build to Firebase"
  lane :dev do
    deploy_android_firebase(
      firebase_app_id: "1:1234567890:android:abcdef",
      app_identifier: "com.example.exampleapp.dev",
      app_name: "Example APP Dev",
      flavor: "development",
      target: "lib/main_development.dart",
      build: "apk",
      slack_message: "Example APP Android: Development build uploaded to Firebase!"
    )
  end
end
```

#### Example Android Appfile

```ruby
# apps/better_iptv/android/fastlane/Appfile

# If not using common configs.rb, configure app-specific credentials here
ENV['JSON_KEY_FILE_PATH'] = "../store_keys/xxx.json"
ENV['SLACK_URL'] = "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX"
```

### Configuration Priority

1. App-specific Appfile configurations take precedence over common configs
2. Environment variables set in the CI/CD pipeline take precedence over both
3. If using common configs.rb, Appfile configurations are optional
