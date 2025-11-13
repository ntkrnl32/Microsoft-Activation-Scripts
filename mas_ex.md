# mas explained

## symbols

```
MAS source code in here
```

> MAS original comments

## actuall part

```
set "_cmdf=%~f0"
```

set `_cmdf` to current full path

```
for %%# in (%*) do (
if /i "%%#"=="re1" set re1=1
if /i "%%#"=="re2" set re2=1
if /i "%%#"=="-qedit" (set re1=1&set re2=1)
)
```

for every param check `re1` if exist set `re1` to 1
for every param check `re2` if exist set `re2` to 1
for every param check `-qedit` if exist set `re1` & `re2` to 1

```
if exist %SystemRoot%\Sysnative\cmd.exe if not defined re1 (
setlocal EnableDelayedExpansion
start %SystemRoot%\Sysnative\cmd.exe /c ""!_cmdf!" %* re1"
exit /b
)
```

if x64 version of cmd.exe exists then relaunch the script using that with `re1` param.
if already relaunched then ignore.

```
if exist %SystemRoot%\SysArm32\cmd.exe if %PROCESSOR_ARCHITECTURE%==AMD64 if not defined re2 (
setlocal EnableDelayedExpansion
start %SystemRoot%\SysArm32\cmd.exe /c ""!_cmdf!" %* re2"
exit /b
)
```

> Re-launch the script with ARM32 process if it was initiated by x64 process on ARM64 Windows

if os is arm64 & running on x64 process relaunch with arm32 cmd.exe

```
set _args=%*
```

set `_args` to args

```
if defined _args set _args=%_args:"=%
if defined _args set _args=%_args:re1=%
if defined _args set _args=%_args:re2=%
```

delete

 - `"`
 - `re1`
 - `re2`

from `_args`

```
for %%A in (%_args%) do (
if /i "%%A"=="-el"                    set _elev=1
)
```

if found `-el` (not case-sensitive) in args set `_elev` to 1

```
if defined _args echo "%_args%" | find /i "/" >nul && set _unattended=1
```

if found `/` in args set `unattended` to 1

```
set "nul1=1>nul"
set "nul2=2>nul"
set "nul6=2^>nul"
set "nul=>nul 2>&1"
```

nul1 = ignore stdout
nul2 = ignore stderr
nul6 = escaped ignore stderr
nul = ignore all (both stdout & stderr)

```
call :dk_setvar
```

call the function `dk_setvar`

```
set ps=%SysPath%\WindowsPowerShell\v1.0\powershell.exe
set psc=%ps% -nop -c
```

set powershell related varibles

```
for /f "tokens=2 delims=[]" %%G in ('ver') do for /f "tokens=2,3,4 delims=. " %%H in ("%%~G") do set "winbuild=%%J"
```

~~what the hell is this~~

1. run `ver`
2. use `[` `]` as delimiters
3. takes token 2 (the part inside the brackets)
4. result in `%%G`: `Version 10.0.26220.7051]`
5. use ` `(space) and `.` as delimiters and takes the 2,3,4 token (`10`,`0`,`26220`) and assign to `%%H`, `%%I`, `%%J`
6. set `winbuild` to the value of `%%J`

```
set _slexe=sppsvc.exe& set _slser=sppsvc
if %winbuild% LEQ 6300 (set _slexe=SLsvc.exe& set _slser=SLsvc)
if %winbuild% LSS 7600 if exist "%SysPath%\SLsvc.exe" (set _slexe=SLsvc.exe& set _slser=SLsvc)
if %_slexe%==SLsvc.exe set _vis=1
```
for default use `sppsvc` (`Windows 8`+)

if build ≤ 6300 use `SLsvc.exe`.

if build < 7600 and the old file actually exists use `SLsvc.exe`.

(%SysPath% is usually %SystemRoot%\System32)

if `_slexe` ended up using the old name `SLsvc.exe` set `_vis` to 1

```
set _NCS=1
if %winbuild% LSS 10586 set _NCS=0
if %winbuild% GEQ 10586 reg query "HKCU\Console" /v ForceV2 %nul2% | find /i "0x0" %nul1% && (set _NCS=0)
```

if build < 10586 force use old console
if build ≥ 10586 check `HKCU\Console\ForceV2`
if value exists and is `0x0` then user disables the new console, set `_NCS` to 0

```
if %_NCS% EQU 1 (
for /F %%a in ('echo prompt $E ^| cmd') do set "esc=%%a"
set     "Red="41;97m""
set    "Gray="100;97m""
set   "Green="42;97m""
set    "Blue="44;97m""
set   "White="107;91m""
set    "_Red="40;91m""
set  "_White="40;37m""
set  "_Green="40;92m""
set "_Yellow="40;93m""
) else (
set     "Red="Red" "white""
set    "Gray="Darkgray" "white""
set   "Green="DarkGreen" "white""
set    "Blue="Blue" "white""
set   "White="White" "Red""
set    "_Red="Black" "Red""
set  "_White="Black" "Gray""
set  "_Green="Black" "Green""
set "_Yellow="Black" "Yellow""
)
```

if `_NCS` = 1:
 - set `esc` to the ESC charactor
 - set new ANSI colors
else:
 - set them to strings so that old consoles dont output rubbish

```
if %~z0 GEQ 200000 (
set "_exitmsg=Go back"
set "_fixmsg=Go back to Main Menu, select Troubleshoot and run Fix Licensing option."
) else (
set "_exitmsg=Exit"
set "_fixmsg=In MAS folder, run Troubleshoot script and select Fix Licensing option."
)
exit /b
```

`%~z0`: the size of the current script

if ≥ 200000 bytes then decides this is the AIO file

else decides this is a seperate file

then exits `dk_setvar` function

```
if %winbuild% EQU 1 (
%eline%
echo Failed to detect Windows build number.
echo:
setlocal EnableDelayedExpansion
set fixes=%fixes% %mas%troubleshoot
call :dk_color2 %Blue% "Check this webpage for help - " %_Yellow% " %mas%troubleshoot"
goto dk_done
)
```

If `winbuild` = 1 decides winbuild detect failed, output some help info and goto `dk_done`

```
:dk_done

echo:
if %_unattended%==1 timeout /t 2 & exit /b

if defined fixes (
call :dk_color %White% "Follow ALL the ABOVE blue lines.   "
call :dk_color2 %Blue% "Press [1] to Open Support Webpage " %Gray% " Press [0] to Ignore"
choice /C:10 /N
if !errorlevel!==2 exit /b
if !errorlevel!==1 (start %selfgit% & start %github% & for %%# in (%fixes%) do (start %%#))
)

if defined terminal (
call :dk_color %_Yellow% "Press [0] key to %_exitmsg%..."
choice /c 0 /n
) else (
call :dk_color %_Yellow% "Press any key to %_exitmsg%..."
pause %nul1%
)

exit /b
```

if `fixes` defined:
- press 0 -> exit function
- press 1 -> open support pages:
  - %github%
  - %selfgit%
  - every url stored in `fixes`

if `terminal` defined:
- press 0 to exit the function

else:
- press any key to exit funtion

```
set "_work=%~dp0"
if "%_work:~-1%"=="\" set "_work=%_work:~0,-1%"
```

set `_work` to folder containing the batch file

if `_work` ends with `\` (which always does) removes that

```
set "_batf=%~f0"
set "_batp=%_batf:'=''%"
```

set `_batf` to the full absolute batch file path

replace `'` with `''` (for powershell escaping)

```
set _PSarg="""%~f0""" -el %_args%
set _PSarg=%_PSarg:'=''%
```

set `_PSarg` to the current absolute path + `-el` + value of `_args`

replace `'` with `''` (for powershell escaping)

set `_ttemp` to the current user temp folder

```
echo "!_batf!" | find /i "!_ttemp!" %nul1% && (
if /i not "!_work!"=="!_ttemp!" (
%eline%
echo The script was launched from the temp folder.
echo You are most likely running the script directly from the archive file.
echo:
echo Extract the archive file and launch the script from the extracted folder.
goto dk_done
)
)
```
if found the current file in temp folder then output error messages and exit.

```
%nul1% fltmc || (
if not defined _elev %psc% "start cmd.exe -arg '/c \"!_PSarg!\"' -verb runas" && exit /b
%eline%
echo This script needs admin rights.
echo Right click on this script and select 'Run as administrator'.
goto dk_done
)
```

tests admin rights with `fltmc`(which only works with admin)

if `_elev` is not defined (no `-el` augument) & check failed re-run the script using PowerShell admin rights &
output error message and exit

```
::pstst $ExecutionContext.SessionState.LanguageMode :pstst
```

**This line looks like a comment (`::`), but it is deliberately placed inside the batch file so that a PowerShell script can read the batch file itself and split it at `:pstst`.**

**Everything after the second `:pstst` is pure PowerShell code that will be executed later.**

```
for /f "delims=" %%a in ('%psc% "if ($PSVersionTable.PSEdition -ne 'Core') {
    $f=[System.IO.File]::ReadAllText('!_batp!') -split ':pstst';
    . ([scriptblock]::Create($f[1]))
}" %nul6%') do (set tstresult=%%a)
```

%psc% = `powershell -nop -c` (no profile, no window)

Only runs on Windows PowerShell (not PowerShell Core) because Core cannot read the batch file the same way on all systems.

Reads the entire current batch file (`!_batp!` = escaped full path)

Splits it at `:pstst`

Takes the second part (`$f[1]`) → that’s exactly the hidden line `$ExecutionContext.SessionState.LanguageMode`

Executes it with `[scriptblock]::Create()`

Captures the output (`FullLanguage` or `ConstrainedLanguage`) into `tstresult`

```
if /i not "%tstresult%"=="FullLanguage" (
    for /f ... ('%psc% "$ExecutionContext.SessionState.LanguageMode"') do set tstresult2=%%a
    echo Test 1 - %tstresult%
    echo Test 2 - !tstresult2!
    echo:
)
```

If its in `ConstrainedLanguage` print the results.

```
echo: !tstresult2! | findstr /i "ConstrainedLanguage RestrictedLanguage NoLanguage" %nul1% && (
echo FullLanguage mode not found in PowerShell. Aborting...
echo If you have applied restrictions on Powershell then undo those changes.
echo:
set fixes=%fixes% %mas%fix_powershell
call :dk_color2 %Blue% "Check this webpage for help - " %_Yellow% " %mas%fix_powershell"
goto dk_done
)
```

If its in `ConstrainedLanguage` mode (not `FullLanguage` mode) outputs the error, set `fixes` and exit.

```
cmd /c "%psc% "$PSVersionTable.PSEdition"" | find /i "Core" %nul1% && (
echo Windows Powershell is needed for MAS but it seems to be replaced with Powershell core. Aborting...
echo:
set fixes=%fixes% %mas%in-place_repair_upgrade
call :dk_color2 %Blue% "Check this webpage for help - " %_Yellow% " %mas%in-place_repair_upgrade"
goto dk_done
)
```

> check Powershell core version

```
for /r "%ProgramFiles%\" %%f in (secureboot.exe) do if exist "%%f" (
echo "%%f"
echo Mal%blank%ware found, PowerShell is not working properly.
echo:
set fixes=%fixes% %mas%remove_mal%w%ware
call :dk_color2 %Blue% "Check this webpage for help - " %_Yellow% " %mas%remove_mal%w%ware"
goto dk_done
)
```

> check for Mal-ware that may cause issues with Powershell

```
if /i "!tstresult2!"=="FullLanguage" (
    cmd /c "%psc% ""try {[System.AppDomain]::CurrentDomain.GetAssemblies(); [System.Math]::Sqrt(144)} catch {Exit 3}""" %nul%
    if !errorlevel!==3 (
        echo Windows Powershell failed to load .NET command. Aborting...
        echo:
        set fixes=%fixes% %mas%in-place_repair_upgrade
        call :dk_color2 %Blue% "Check this webpage for help - " %_Yellow% " %mas%in-place_repair_upgrade"
        goto dk_done
    )
)
```

> check if .NET is working properly

only run this extra test if the previous language-mode test passed, check some regular `.NET` features

if exits with code 3, output error messages, set `fixes` and exit


