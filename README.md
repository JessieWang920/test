@echo off
setlocal EnableDelayedExpansion


echo [1] find NodeRED PID
for /f "tokens=5" %%P in ('netstat -ano ^| findstr /r /c:":1880 .*LISTENING"') do set NODEPID=%%P
echo   → NODEPID=!NODEPID!

echo [2] find NODEPID's parent PID 
set "PARENTPID="
for /f %%A in ('
    powershell -Command "Get-CimInstance Win32_Process -Filter 'ProcessId = !NODEPID!' | Select-Object -ExpandProperty ParentProcessId"
') do (
    set /a "PARENTPID=%%A"
)
echo   → PARENTPID=!PARENTPID!


if defined PARENTPID (
    echo [3] kill all PID !PARENTPID! 
    taskkill /F /T /PID !PARENTPID!

    echo [4] waiting 2 sec
    timeout /t 2 >nul
  
) else (
    echo   [ERROR] 無法取得 Parent PID
)


echo [5] start NodeRED
start "" "%APPDATA%\npm\node-red.cmd"

endlocal
