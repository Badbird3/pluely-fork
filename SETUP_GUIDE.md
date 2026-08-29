# Pluely Build Setup Guide for Windows

This guide will help you install all prerequisites needed to build Pluely on Windows.

---

## Prerequisites Checklist

- [ ] Node.js v18+
- [ ] Rust (latest stable)
- [ ] Visual Studio C++ Build Tools
- [ ] WebView2 Runtime

---

## Step 1: Install Rust

### Option A: Using rustup-init.exe (Recommended)

1. **Download Rust installer:**
   - Visit: https://rustup.rs/
   - Or direct download: https://win.rustup.rs/x86_64

2. **Run the installer:**
   ```powershell
   # Download and run (PowerShell as Administrator)
   Invoke-WebRequest -Uri "https://win.rustup.rs/x86_64" -OutFile "$env:TEMP\rustup-init.exe"
   & "$env:TEMP\rustup-init.exe"
   ```

3. **Follow the prompts:**
   - Press `1` for default installation
   - Wait for installation to complete

4. **Restart your terminal** (PowerShell/CMD) to refresh PATH

5. **Verify installation:**
   ```powershell
   rustc --version
   cargo --version
   ```

   Expected output:
   ```
   rustc 1.xx.x (...)
   cargo 1.xx.x (...)
   ```

---

## Step 2: Install Microsoft C++ Build Tools

Rust requires the Microsoft C++ build tools for linking.

### Option A: Visual Studio Build Tools (Recommended)

1. **Download:**
   - Visit: https://visualstudio.microsoft.com/downloads/
   - Scroll to "All Downloads" → "Tools for Visual Studio"
   - Download "Build Tools for Visual Studio 2022"

2. **Install with required components:**
   - Run the installer
   - Select "Desktop development with C++"
   - Make sure these are checked:
     - MSVC v143 - VS 2022 C++ x64/x86 build tools
     - Windows 11 SDK (or Windows 10 SDK)
     - C++ CMake tools for Windows

3. **Restart your computer** after installation

### Option B: Quick Install (Alternative)

If you already have Visual Studio installed, ensure you have the C++ workload:
- Open Visual Studio Installer
- Click "Modify" on your installation
- Check "Desktop development with C++"
- Install

---

## Step 3: Install WebView2 (Usually Pre-installed)

Most Windows 11 systems have this pre-installed. To check:

```powershell
# Check if WebView2 is installed
Get-ItemProperty -Path "HKLM:\SOFTWARE\WOW6432Node\Microsoft\EdgeUpdate\Clients\{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}" -Name pv
```

If not installed:
1. Download from: https://developer.microsoft.com/en-us/microsoft-edge/webview2/
2. Install "Evergreen Bootstrapper"

---

## Step 4: Verify Node.js

```powershell
node --version
npm --version
```

Expected output:
```
v18.x.x (or higher)
9.x.x (or higher)
```

If not installed:
1. Download from: https://nodejs.org/
2. Install LTS version (v18 or higher)
3. Restart terminal

---

## Step 5: Install Pluely Dependencies

```powershell
cd C:\Users\badbi\Documents\pluely\pluely
npm install
```

---

## Step 6: Build Pluely

```powershell
# Development build (faster, larger)
npm run tauri dev

# Production build (optimized, smaller)
npm run tauri build
```

### Build Output Locations

After successful build, installers will be in:

**MSI Installer:**
```
src-tauri\target\release\bundle\msi\
```

**NSIS Installer:**
```
src-tauri\target\release\bundle\nsis\
```

**Executable (no installer):**
```
src-tauri\target\release\pluely.exe
```

---

## Common Issues & Solutions

### Issue: "program not found" or "cargo not found"

**Solution:**
1. Restart your terminal after installing Rust
2. Verify PATH includes: `C:\Users\{YourUser}\.cargo\bin`
3. Run in PowerShell:
   ```powershell
   $env:Path += ";$env:USERPROFILE\.cargo\bin"
   ```

### Issue: "link.exe not found" or "MSVC not found"

**Solution:**
1. Install Visual Studio Build Tools (see Step 2)
2. Restart your computer
3. Verify installation:
   ```powershell
   & "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64
   ```

### Issue: "failed to run custom build command for..."

**Solution:**
1. Make sure you're running as Administrator
2. Close and reopen your terminal
3. Clear cargo cache:
   ```powershell
   cargo clean
   npm run tauri build
   ```

### Issue: Build is very slow

**Solution:**
- First build takes 10-30 minutes (normal)
- Subsequent builds are much faster (~2-5 minutes)
- Use `npm run tauri dev` for development (faster iterations)

---

## Quick Setup Script (Run as Administrator)

Save this as `setup.ps1` and run:

```powershell
# setup.ps1 - Run as Administrator
Write-Host "Installing Pluely prerequisites..." -ForegroundColor Green

# Check Node.js
try {
    $nodeVersion = node --version
    Write-Host "✓ Node.js $nodeVersion found" -ForegroundColor Green
} catch {
    Write-Host "✗ Node.js not found - Please install from https://nodejs.org/" -ForegroundColor Red
    exit 1
}

# Install Rust
if (-not (Get-Command rustc -ErrorAction SilentlyContinue)) {
    Write-Host "Installing Rust..." -ForegroundColor Yellow
    Invoke-WebRequest -Uri "https://win.rustup.rs/x86_64" -OutFile "$env:TEMP\rustup-init.exe"
    & "$env:TEMP\rustup-init.exe" -y
    $env:Path += ";$env:USERPROFILE\.cargo\bin"
    Write-Host "✓ Rust installed" -ForegroundColor Green
} else {
    $rustVersion = rustc --version
    Write-Host "✓ Rust $rustVersion found" -ForegroundColor Green
}

# Check MSVC
$vsWhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
if (Test-Path $vsWhere) {
    $vsInstall = & $vsWhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
    if ($vsInstall) {
        Write-Host "✓ Visual Studio C++ Build Tools found" -ForegroundColor Green
    } else {
        Write-Host "✗ Visual Studio C++ Build Tools not found" -ForegroundColor Red
        Write-Host "  Please install from: https://visualstudio.microsoft.com/downloads/" -ForegroundColor Yellow
    }
} else {
    Write-Host "✗ Visual Studio not found" -ForegroundColor Red
    Write-Host "  Please install Build Tools from: https://visualstudio.microsoft.com/downloads/" -ForegroundColor Yellow
}

# Install npm dependencies
Write-Host "`nInstalling npm dependencies..." -ForegroundColor Yellow
cd "C:\Users\badbi\Documents\pluely\pluely"
npm install

Write-Host "`n✓ Setup complete!" -ForegroundColor Green
Write-Host "You can now run: npm run tauri build" -ForegroundColor Cyan
```

---

## Build Times Reference

| Build Type | First Time | Subsequent |
|------------|-----------|------------|
| Dev (`tauri dev`) | 15-30 min | 2-5 min |
| Release (`tauri build`) | 20-40 min | 5-10 min |

*Times vary based on CPU speed and available RAM*

---

## After Successful Build

Your custom Pluely build will be ready to install:

1. Navigate to `src-tauri\target\release\bundle\msi\`
2. Double-click the `.msi` file
3. Follow installation wizard
4. Launch Pluely from Start Menu or Desktop

---

## Testing Changes

To test your modifications without creating an installer:

```powershell
# Run in development mode
npm run tauri dev
```

This will:
- Compile and run the app immediately
- Show console logs for debugging
- Auto-reload on code changes (frontend only)
- Faster iteration for testing

---

## Need Help?

**Official Tauri Prerequisites:**
https://v2.tauri.app/start/prerequisites/

**Rust Installation:**
https://www.rust-lang.org/tools/install

**Visual Studio Build Tools:**
https://visualstudio.microsoft.com/downloads/

**Pluely Issues:**
https://github.com/iamsrikanthnani/pluely/issues

---

Generated: 2026-06-23
