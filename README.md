<div align="center">
  <img src="OnscreenTranslator/Resources/Assets.xcassets/AppIcon.appiconset/AppIcon-180.png" width="112" alt="Onscreen Translator app icon">
  <h1>Onscreen Translator</h1>
  <p><strong>Translate what is on your iPhone screen with a Shortcut.</strong></p>
  <p>
    <img alt="Platform: iOS 18+" src="https://img.shields.io/badge/iOS-18%2B-000000?logo=apple">
    <img alt="Swift 5" src="https://img.shields.io/badge/Swift-5-F05138?logo=swift&logoColor=white">
    <img alt="Built with SwiftUI" src="https://img.shields.io/badge/UI-SwiftUI-0D96F6">
    <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-green.svg"></a>
  </p>
</div>

Onscreen Translator turns a screenshot into a translated image while preserving
the original layout. Trigger it from any app with the Action Button or Back Tap,
or choose an image manually inside the app.

It combines Apple Vision OCR, Apple Translation, App Intents, and a lightweight
renderer that replaces recognized text in place. No screen recording, jailbreak,
or private iOS APIs are required.

## See it in action

<table>
  <tr>
    <th width="50%">Before — Thai</th>
    <th width="50%">After — Simplified Chinese</th>
  </tr>
  <tr>
    <td><img src="docs/screenshots/weather-before.png" alt="Thai weather warning before translation"></td>
    <td><img src="docs/screenshots/weather-after.png" alt="Weather warning translated into Simplified Chinese"></td>
  </tr>
</table>

<p align="center">
  <img src="docs/screenshots/cosme-after.png" width="31%" alt="Japanese cosmetics post translated into Simplified Chinese">
  &nbsp;
  <img src="docs/screenshots/skin-analyzer-after.png" width="31%" alt="Japanese skin analyzer translated into Simplified Chinese">
</p>

The examples above are real renderer output. Results vary with text size,
contrast, typography, and the language pair.

## Highlights

- **Translate from any app** through the `Translate Current Screenshot` App Intent.
- **Preserve the screen layout** by masking OCR regions and drawing translations
  back into their original positions.
- **Prefer on-device processing** with Apple Vision and Apple Translation.
- **Use a cloud fallback** through an OpenAI-compatible chat-completions endpoint
  when an on-device language pair is unavailable.
- **Handle mixed-language screens** with configurable OCR language hints.
- **Review past results** from the built-in translation history.
- **Keep credentials private** in the iOS Keychain.

## Requirements

- iPhone running iOS 18 or later
- Xcode 16 or later for development
- Apple Translation language models installed for fully on-device translation
- Optional Gemini or other OpenAI-compatible API key for cloud fallback

## Quick start

1. Clone the repository and open `OnscreenTranslator.xcodeproj` in Xcode.
2. Select the **OnscreenTranslator** target.
3. Under **Signing & Capabilities**, choose your Development Team and set a
   unique bundle identifier.
4. Build and run on an iPhone.
5. In the app's **Settings** tab, choose a target language. Add a cloud API key
   only if you want fallback translation.

The checked-in Xcode project is ready to open. If you change `project.yml`,
regenerate it with [XcodeGen](https://github.com/yonaskolb/XcodeGen):

```sh
xcodegen generate
```

## Create the Shortcut

Create a Shortcut named **Translate Screen** with these three actions:

1. **Take Screenshot**
2. **Translate Current Screenshot**
   - Pass the screenshot from step 1 to **Screenshot**.
   - Optionally choose a target language; leave it blank to use the app setting.
3. **Quick Look**
   - Pass the translated file from step 2 to Quick Look.

Then assign it to one of these triggers:

- **Action Button:** Settings → Action Button → Shortcut
- **Back Tap:** Settings → Accessibility → Touch → Back Tap
- **Manual:** open Onscreen Translator and tap **Choose Screenshot**

When triggered, iOS captures the visible screen, the App Intent recognizes and
translates its text, and Quick Look presents the rendered image.

> [!NOTE]
> iOS does not allow regular App Store apps to read or draw over another app's
> live interface. This project uses the screenshot-and-Quick-Look flow as the
> closest public-API alternative.

## Translation settings

The defaults are ready for Thai, Simplified Chinese, English, Japanese, and
Korean OCR. You can change:

| Setting | Default |
| --- | --- |
| Target language | Chinese (`zh-Hans`) |
| Prefer on-device translation | Enabled |
| Cloud endpoint | Gemini OpenAI-compatible endpoint |
| Cloud model | `gemini-2.5-flash` |
| OCR language hints | `th-TH, zh-Hans, en-US, ja-JP, ko-KR` |

To provide a development-only Gemini key at build time, create
`OnscreenTranslator/Config/Secrets.xcconfig`:

```xcconfig
GEMINI_API_KEY = your_key_here
```

That file is ignored by Git. You can also enter the key in the app, where it is
stored in Keychain.

## How it works

```text
Screenshot
    ↓
Apple Vision OCR
    ↓
Language filtering and paragraph grouping
    ↓
Apple Translation ── or ── OpenAI-compatible fallback
    ↓
Background-aware text replacement
    ↓
Translated PNG
```

The OCR pass scans the entire screenshot, including text inside photos, banners,
and promotional cards. Stylized or low-contrast text is naturally harder than
native UI text.

## Build from the command line

```sh
xcodebuild \
  -project OnscreenTranslator.xcodeproj \
  -scheme OnscreenTranslator \
  -destination 'generic/platform=iOS' \
  -derivedDataPath ./DerivedData \
  build CODE_SIGNING_ALLOWED=NO
```

## Build an unsigned IPA with GitHub Actions

The **Build Unsigned IPA** workflow runs on pushes and pull requests, or
manually from the repository's **Actions** tab with **Run workflow**. After a
successful run, download the `OnscreenTranslator-unsigned-ipa` artifact from
the workflow run. It contains `OnscreenTranslator-unsigned.ipa`.

The workflow builds for a generic iOS device using the Xcode version on the
GitHub-hosted macOS runner. The resulting IPA is unsigned and must be signed
with a valid certificate and provisioning profile before it can be installed
on an iPhone.

## Project structure

```text
OnscreenTranslator/
├── Sources/       SwiftUI app, OCR, translation, and rendering
├── Resources/     Info.plist and app icon
├── Tests/         Translation pipeline tests and fixtures
└── Config/        Shared Xcode configuration
Tools/
└── RenderPreview/ macOS command-line renderer preview
```

## Contributing

Issues and focused pull requests are welcome. When reporting OCR or rendering
problems, include the source screenshot, target language, and a description of
the expected result. Remove personal or sensitive information before uploading
screenshots.

## License

Onscreen Translator is available under the [MIT License](LICENSE).
