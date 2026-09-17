# CursorCloak Maintenance & Developer Guide

> **"The Project Bible"** - This document is the single source of truth for maintaining, developing, and releasing CursorCloak.

## 1. Project Architecture

### **Core Components**
- **`App.xaml.cs`**: Entry point. Handles:
  - Single Instance Enforcement (`Mutex`)
  - Administrator Privilege Check
  - Global Exception Handling
  - Global Mouse Hook Registration
- **`MainWindow.xaml.cs`**: Main UI logic. *Do not put business logic here.*
- **`Services/`**:
  - **`CursorEngine.cs`**: Low-level P/Invoke calls to hide/show cursors. **CRITICAL**: Must handle GDI object disposal.
  - **`HotKeyManager.cs`**: Registers system-wide hotkeys (`Alt+H`, `Alt+S`).
  - **`GlobalMouseHook.cs`**: Low-level Windows Hook (`WH_MOUSE_LL`) for auto-hide functionality.
  - **`StartupManager.cs`**: Registry manipulation for "Start with Windows".
  - **`SettingsManager.cs`**: JSON serialization for user preferences.
- **`Models/`**: Data structures (`Settings`, `UserConfig`).

### **Critical Rules**
1.  **GDI Resources**: Any `IntPtr` from `CreateIcon`, `CreateBitmap`, or `LoadCursor` MUST be released using `DestroyIcon` or `DeleteObject`.
2.  **Administrator**: The app requires Admin privileges to set system cursors. This is enforced in `app.manifest` and `App.xaml.cs`.
3.  **Single Instance**: Enforced via named Mutex `CursorCloak_SingleInstance_Mutex`.

---

## 2. Release Process

### **Step 1: Update Version Numbers**
Run `.\scripts\update-version.ps1 -NewVersion "X.Y.Z" -ReleaseType "Release Name"` to update
version references across the project, including:
1.  `src/CursorCloak.UI/CursorCloak.UI.csproj` (AssemblyVersion, FileVersion)
2.  `src/CursorCloak.UI/app.manifest`
3.  `.github/workflows/build-release.yml` (PROJECT_VERSION)
4.  `scripts/setup.iss`, `scripts/setup-selfcontained.iss`
5.  `build.ps1` (File paths)
6.  `README.md` (Download links)

Then run `.\scripts\verify-version.ps1 -ExpectedVersion "X.Y.Z"` to confirm consistency.

### **Step 2: Documentation**
1.  **`CHANGELOG.md`**: Add new entry at top.
2.  **`docs/VERSION.md`**: Move current to history, add new top section.

### **Step 3: Git Workflow**
1.  Commit: `git commit -am "chore: Release v2.0.1"`
2.  Push: `git push origin main`
3.  Tag: `git tag v2.0.1`
4.  Push Tag: `git push origin v2.0.1` -> **Triggers GitHub Action**

---

## 3. Coding Standards

- **Style**: Follow `.editorconfig`.
- **Comments**: XML Documentation (`///`) on ALL public members.
- **Safety**: Wrap all P/Invoke calls in `try-catch` where appropriate.
- **Cleanup**: Implement `IDisposable` for classes holding native handles.

## 4. Common Pitfalls & Fixes

- **"Project is a hell"**: Usually means version mismatch or GDI leaks.
  - **Fix**: Run version audit. Check Task Manager for GDI object count.
- **Tray Icon Stuck**: App crashed without `NotifyIcon.Dispose()`.
  - **Fix**: Ensure `App.xaml.cs` handles `DispatcherUnhandledException` and disposes icon.
- **Cursor Not Hiding**: Admin privileges missing or another app is overriding hooks.

---

## 5. Future Roadmap
- [ ] Add Unit Tests for `SettingsManager` and other Services classes (see `tests/README.md`).
- [ ] Multi-monitor support (per-screen cursor hiding settings).
- [ ] Custom hotkey configuration UI.
