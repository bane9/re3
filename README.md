# re3 — Build & Run Guide (Windows, 64-bit OpenGL)

This is a step-by-step guide to build this fork of **re3** from scratch on Windows and run it
on top of a Steam copy of GTA III. It targets the **64-bit OpenGL** build
(`win-amd64-librw_gl3_glfw-oal`), which is the configuration this fork is developed and tested against.

> Looking for the original upstream readme (features, other platforms, contributing, etc.)?
> See [README-old.md](README-old.md).

---

## 0. What you need

- **Windows 10 or 11 (64-bit).**
- **Git** — https://git-scm.com/download/win
- **Visual Studio 2022 Community** (free) — https://visualstudio.microsoft.com/vs/community/
  - During install, select the **"Desktop development with C++"** workload.
  - That workload includes MSBuild, the MSVC compiler, and a Windows SDK — everything needed to compile.
- **A legitimate copy of GTA III on Steam** (used only for the original game data files).

Total disk needed: a few GB for Visual Studio + the repo + the game.

---

## 1. Clone the repository (with submodules)

The renderer (`librw`) and audio codecs are **git submodules**, so you must clone **recursively**.
Open *Git Bash* or a terminal and run:

```sh
git clone --recursive https://github.com/bane9/re3.git
cd re3
```

If you already cloned without `--recursive`, pull the submodules in:

```sh
git submodule update --init --recursive
```

> If `vendor/librw` is empty, the build will fail with `Cannot open include file: 'rw.h'`.
> That always means the submodules weren't checked out — run the command above.

---

## 2. Download GLFW (required for the OpenGL build)

The OpenGL build links against a prebuilt **GLFW 3.3.2** for windowing/input. It is **not** a
submodule, so download it manually:

1. Download **`glfw-3.3.2.bin.WIN64.zip`**:
   https://github.com/glfw/glfw/releases/download/3.3.2/glfw-3.3.2.bin.WIN64.zip
2. Extract it into the `vendor/` folder so that this exact path exists:

   ```
   re3/vendor/glfw-3.3.2.bin.WIN64/include/GLFW/glfw3.h
   re3/vendor/glfw-3.3.2.bin.WIN64/lib-vc2019/glfw3.lib
   ```

   (i.e. the folder inside `vendor/` must be named exactly `glfw-3.3.2.bin.WIN64`.)

> If this is missing, the build fails with `Cannot open include file: 'GLFW/glfw3.h'`.

---

## 3. Generate the Visual Studio solution (premake)

From the repo root, run the premake helper for VS2019 project format:

```
premake-vs2019.cmd
```

This runs `premake5 vs2019 --with-librw` and generates the solution into the **`build/`** folder
(`build/re3.sln`), including the `librw` renderer project.

> Re-run this whenever you pull changes that add/remove source files.

---

## 4. Open and "migrate" the solution in Visual Studio

1. Open **`build/re3.sln`** in Visual Studio 2022.
2. The projects were generated for the VS2019 toolset (v142). VS2022 will offer to upgrade them:
   - If you see a **"Retarget Projects"** dialog on open, accept it.
   - Otherwise, right-click the **solution** in Solution Explorer → **Retarget solution**.
   - Choose the **latest installed Platform Toolset** and **Windows SDK version**, then **OK**.

   This is the "migration" step — it just points the projects at your installed compiler/SDK.

---

## 5. Select the build configuration

At the top of Visual Studio, set the two dropdowns to:

- **Configuration:** `Release`
- **Solution Platform:** `win-amd64-librw_gl3_glfw-oal`

This is the 64-bit OpenGL + GLFW + OpenAL build.

---

## 6. Install GTA III from Steam

1. In Steam, install **Grand Theft Auto III**.
2. Note its install folder. By default it is:

   ```
   C:\Program Files (x86)\Steam\steamapps\common\Grand Theft Auto 3
   ```

   (Right-click the game in Steam → *Manage* → *Browse local files* to confirm.)

You do **not** run the game from Steam — you'll run the compiled `re3.exe` from that folder instead.

---

## 7. Build (Release)

In Visual Studio: **Build → Build Solution** (or press **F7** / **Ctrl+Shift+B**).

The first build also compiles `librw` and takes a little while. When it finishes, the executable is at:

```
re3/bin/win-amd64-librw_gl3_glfw-oal/Release/re3.exe
```

---

## 8. Copy the build + game files into the Steam folder

Copy the following **into the Steam GTA III folder**
(`C:\Program Files (x86)\Steam\steamapps\common\Grand Theft Auto 3`):

1. The compiled executable:
   ```
   bin/win-amd64-librw_gl3_glfw-oal/Release/re3.exe
   ```
2. The **entire contents** of the repo's **`gamefiles/`** folder (merge/overwrite into the Steam
   folder — it adds `data/`, `models/`, `TEXT/`, `neo/`, `gamecontrollerdb.txt`, etc. that this fork needs).
3. The three audio runtime DLLs (required by the OpenAL build):
   ```
   vendor/openal-soft/dist/Win64/OpenAL32.dll
   vendor/mpg123/dist/Win64/libmpg123-0.dll
   vendor/libsndfile/dist/Win64/libsndfile-1.dll
   ```

After copying, the Steam folder should contain `re3.exe`, the three DLLs, and the original
game's `models/`, `data/`, etc. (now overlaid with the files from `gamefiles/`).

---

## 9. Run

Launch **`re3.exe`** from the Steam GTA III folder (double-click it, or make a shortcut).

That's it — the game runs on the freshly compiled build.

---

## Quick reference (TL;DR)

```sh
git clone --recursive https://github.com/bane9/re3.git
cd re3
# extract glfw-3.3.2.bin.WIN64.zip into vendor/  -> vendor/glfw-3.3.2.bin.WIN64/
premake-vs2019.cmd
# open build/re3.sln in VS2022, retarget when prompted
# set Configuration=Release, Platform=win-amd64-librw_gl3_glfw-oal, then Build (F7)
# copy bin/win-amd64-librw_gl3_glfw-oal/Release/re3.exe
#  + everything in gamefiles/
#  + OpenAL32.dll, libmpg123-0.dll, libsndfile-1.dll (from vendor/.../dist/Win64)
# into  C:\Program Files (x86)\Steam\steamapps\common\Grand Theft Auto 3
# run re3.exe
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Cannot open include file: 'rw.h'` | Submodules not checked out — run `git submodule update --init --recursive`. |
| `Cannot open include file: 'GLFW/glfw3.h'` | GLFW not extracted to `vendor/glfw-3.3.2.bin.WIN64/` (step 2). |
| `LNK1181: cannot open input file 'rw.lib'` | `librw` didn't build — re-run `premake-vs2019.cmd`, then rebuild the whole solution. |
| Build errors after pulling new code | Re-run `premake-vs2019.cmd` to regenerate the projects. |
| Game starts then closes / no audio | Make sure `re3.exe`, the 3 DLLs, and the `gamefiles/` contents are all in the Steam folder. |
| Crash on launch about game data | The Steam GTA III data must be present in the same folder (install it via Steam first). |
