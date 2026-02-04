
# Abaqus 2023 + Intel oneAPI HPC Toolkit (2023.1) + Visual Studio 2022 — Subroutine Setup (Windows)

This repository documents a repeatable setup to run Abaqus user subroutines (e.g., **UMAT**, **VUMAT**, **DFLUX**) on Windows with:

- **Abaqus 2023** (installation follows EPFL STI-IT instructions)
- **Intel oneAPI HPC Toolkit 2023.1** (offline installer in this repo)
- **Visual Studio Community 2022** (recommended)

This workflow follows the same as the FEAMASTER linking tutorial (environment initialization + Abaqus verify).  
- Guide: https://feamaster.com/link-abaqus-2024-with-fortran/  
- Video: https://www.youtube.com/watch?v=5mVyTES4HeM&t=118s

> Notes  
> - Here we use **Abaqus 2023** (not Abaqus 2024).  
> - This repo includes the oneAPI 2023 installer: **`w_HPCKit_p_2023.1.0.46357.exe`**.

---

## 1) Install Abaqus 2023 (EPFL STI-IT procedure)

Follow the EPFL STI-IT page for Abaqus 2023 installation:
- EPFL Abaqus HOWTO: https://www.epfl.ch/schools/sti/it/abaqus/  (includes the Windows installation recording and EPFL-specific install path / license notes) 

---

## 2) Install Visual Studio Community 2022

Download & install **Visual Studio Community** from Microsoft:
- Visual Studio Community page: https://visualstudio.microsoft.com/vs/community/ 
- (Alternative downloads hub): https://visualstudio.microsoft.com/downloads/ 

### Required workload
During installation, enable:
- **Desktop development with C++**

This is required for Abaqus + Intel Fortran toolchain integration (linking and compilation).

---

## 3) Install Intel oneAPI HPC Toolkit 2023.1 (from this repo)

Run the included installer:
- `w_HPCKit_p_2023.1.0.46357.exe`

Recommended:
- Install with default options unless your IT policy requires otherwise.
- Reboot after installation.

---

## 4) Verify Intel Fortran integration

Open **Visual Studio 2022** → *Continue without code* → *File > New > File*  
Confirm Intel Fortran is present (templates / integration). If not:
- Repair oneAPI installation
- Ensure C++ workload is installed in Visual Studio 

---

## 5) Ensure environment initialization for Abaqus Command

Abaqus must launch with both:
- Visual Studio C++ build environment
- Intel oneAPI compiler environment

### A) Add compiler scripts to PATH (system environment)

**System Properties** → **Environment Variables** → (System) **Path** → add:

- Intel oneAPI compiler environment:
  - `C:\Program Files (x86)\Intel\oneAPI\compiler\latest\env\vars.bat`
- Visual Studio build environment:
  - `C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat`


---

## 6) Edit Abaqus 2023 command launcher (abq2023.bat)

Locate the Abaqus command script for your installation (commonly under SIMULIA “Commands” folder).  
Example (may differ depending on your EPFL installation method):
- `C:\SIMULIA\Commands\abq2023.bat`

1. Make a backup: `abq2023.bat.bak`
2. Open `abq2023.bat` with a text editor
3. After `@echo off` / `setlocal`, add:

```bash
call "C:\Program Files (x86)\Intel\oneAPI\compiler\latest\env\vars.bat" intel64 vs2022
call "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"
```

This forces Abaqus Command to always start with the correct compiler environment.

## 7) Modify the Abaqus shortcut **Target** (Reccomend watching the video (Step 5))

In the video, instead of only editing `abq2023.bat`, you can **link the compiler at the shortcut level** by changing the **Target** field of the Abaqus shortcut (Abaqus Command / Abaqus CAE).  
This ensures the Intel oneAPI + Visual Studio environment is loaded every time you launch Abaqus from the Start Menu.

### 7.1 Locate the Abaqus 2023 shortcut
1. Open **Windows Start Menu**
2. Find **Abaqus 2023** (Abaqus Command or Abaqus CAE)
3. Right-click → **More** → **Open file location**
4. In the folder that opens, right-click the shortcut (e.g., **Abaqus Command**) → **Properties**

### 7.2 Edit the **Target** field (Abaqus Command)

#### (A) Copy the original target (backup)
In **Properties → Shortcut → Target**, copy the original text somewhere (e.g., into a note) so you can revert anytime.

#### (B) Replace Target with a compiler-initialized target
Use your Intel oneAPI compiler env script first, then call the original Abaqus command:

**Recommended Target (Abaqus Command):**
```bash
"C:\Program Files (x86)\Intel\oneAPI\compiler\latest\env\vars.bat" intel64 vs2022 & C:\Windows\System32\cmd.exe /k
```

### 7.3 Edit the Target field (Abaqus CAE)
If you want Abaqus/CAE to launch with the same compiler environment, edit the Abaqus CAE shortcut similarly:
```bash
"C:\Program Files (x86)\Intel\oneAPI\compiler\latest\env\vars.bat" intel64 vs2022 & abq2023 cae
```


## 8) Verify Abaqus can compile user subroutines

Run **Abaqus Verification** , then if you can check following texts in the command prompt, the subroutine set up is successfully completed.

```bash
------------------------------------------------------------

Verify test : Abaqus/Standard with user subroutines verification

     result : PASS

------------------------------------------------------------

Verify test : Abaqus/Explicit with user subroutines verification

  - 'Abaqus/Explicit single precision user subroutine' completed, validation Succeeded

  - 'Abaqus/Explicit double precision user subroutine' completed, validation Succeeded

     result : PASS

------------------------------------------------------------
```

