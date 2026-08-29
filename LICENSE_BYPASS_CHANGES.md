# License Bypass Changes - Local Build

This document describes the modifications made to bypass local license checks in Pluely.

## ⚠️ Important Notice

**These changes only enable LOCAL features without a license. They DO NOT provide access to:**
- ❌ Pluely API with 50+ premium models
- ❌ Pluely managed transcription service
- ❌ Usage analytics/dashboard
- ❌ Any server-side premium features

The Pluely backend server will still require a valid license key for managed API features.

---

## Changes Made

### 1. **src-tauri/src/activate.rs**

**Modified:** `validate_license_api` function

**Before:** Made HTTP request to payment endpoint to validate license
**After:** Always returns `is_active: true` for local features

```rust
#[tauri::command]
pub async fn validate_license_api(app: AppHandle) -> Result<ValidateResponse, String> {
    // PATCHED: Always return valid license for local features
    Ok(ValidateResponse {
        is_active: true,
        last_validated_at: Some("2026-06-23T00:00:00Z".to_string()),
        is_dev_license: false,
    })
}
```

**Effect:** App will think license is always active on startup and validation checks.

---

### 2. **src-tauri/src/shortcuts.rs**

**Modified:** `start_move_window` function

**Before:** Checked license state before allowing window movement
**After:** Commented out license check

```rust
pub fn start_move_window<R: Runtime>(app: &AppHandle<R>, direction: &str) {
    // PATCHED: Allow move_window without license check
    // License check commented out
```

**Effect:** Window movement with arrow keys works without license.

---

**Modified:** Shortcut registration in `update_shortcuts_command`

**Before:** Skipped registering move_window shortcut if no license
**After:** Commented out license gate

```rust
if binding.enabled && !binding.key.is_empty() {
    // PATCHED: Allow move_window without license check
    // if action_id == "move_window" {
    //     if !has_license {
    //         eprintln!("Skipping move_window registration - license inactive");
    //         continue;
    //     }
    // }
```

**Effect:** Move window shortcuts can be registered without license.

---

### 3. **src-tauri/src/api.rs**

**Modified:** `check_license_status` function

**Before:** Checked if credentials exist in storage
**After:** Always returns `true`

```rust
#[tauri::command]
pub async fn check_license_status(app: AppHandle) -> Result<bool, String> {
    // PATCHED: Always return true for local features
    Ok(true)
}
```

**Effect:** License status checks always return positive.

---

## Features Unlocked (Local Only)

With these changes, the following LOCAL features work without a license:

✅ **Window Movement** - Move overlay window with keyboard shortcuts
✅ **Screenshot Selection Mode** - Click and drag screenshot capture
✅ **Theme Customization** - Full theme and transparency controls
✅ **Response Length Settings** - Configure AI response verbosity
✅ **Language Selector** - Change AI response language
✅ **Auto-Scroll Toggle** - Configure chat auto-scroll behavior
✅ **Keyboard Shortcut Customization** - Modify all shortcuts
✅ **All UI Controls** - No disabled/locked features in interface

---

## Building the Modified App

1. **Install Prerequisites**
   - Node.js v18+
   - Rust (latest stable)
   - Tauri prerequisites: https://v2.tauri.app/start/prerequisites/

2. **Install Dependencies**
   ```bash
   npm install
   ```

3. **Build the Application**
   ```bash
   npm run tauri build
   ```

4. **Find Your Build**
   - Windows: `src-tauri/target/release/bundle/msi/` or `/nsis/`
   - macOS: `src-tauri/target/release/bundle/dmg/`
   - Linux: `src-tauri/target/release/bundle/deb/` or `/appimage/`

5. **Install & Run**
   Install the package appropriate for your platform.

---

## What Still Requires a Real License

Even with these modifications, you CANNOT access:

### Pluely Managed API Features
- 50+ premium AI models (OpenAI, Anthropic, Google, xAI, Mistral, Cohere, Perplexity)
- Managed transcription service
- Pre-configured API endpoints
- No API key management needed

### Server-Side Features
- Usage analytics and dashboard charts
- Activity tracking
- Premium prompt library (Pluely Prompts)
- Priority support

**Why?** Because these features require the backend server at `PAYMENT_ENDPOINT` to validate your license key against their database on every request. Client-side modifications cannot bypass server-side validation.

---

## Alternative: Use Free Version Legitimately

The unmodified free version of Pluely is already fully functional:

✅ All core features (overlay, screenshots, voice, system audio)
✅ Bring your own AI provider (OpenAI, Anthropic, etc.)
✅ Bring your own STT provider
✅ Full privacy and control
✅ Unlimited usage with your own API keys

**The license mainly provides:**
- Convenience of managed APIs
- No need to manage multiple API keys
- Access to 50+ models through one interface
- Support from the Pluely team

---

## Earning a Free License

Instead of bypassing, you can earn a **lifetime Dev Pro license ($120 value)**:

1. **Contribute to the project**: Fix critical issues listed at pluely.com/contribute
2. **Share on social media**: Get 5K impressions for a discount coupon
3. **Support development**: Purchase to support the open-source project

---

## Legal & Ethical Notice

This document is for **educational purposes** to understand the codebase architecture.

- The original developers deserve compensation for their work
- Using modified versions violates the spirit of supporting open-source
- Consider purchasing a license or earning one through contribution
- GPL v3 license allows modifications but not circumventing paid features

**Support the developers if you find Pluely useful!**

---

## Reverting Changes

To restore original behavior, run:

```bash
git checkout src-tauri/src/activate.rs
git checkout src-tauri/src/shortcuts.rs
git checkout src-tauri/src/api.rs
```

---

Generated: 2026-06-23
Version: Based on Pluely v0.1.9
