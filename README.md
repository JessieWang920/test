# test
```
@echo off
setlocal EnableDelayedExpansion

rem === 找出佔用 1880 連接埠的 PID ===
echo [1] find NodeRED PID
for /f "tokens=5" %%P in ('netstat -ano ^| findstr /r /c:":1880 .*LISTENING"') do set NODEPID=%%P
echo   → NODEPID=!NODEPID!

rem === 找出其 parent PID ===
echo [2] find NODEPID's parent PID 
set "PARENTPID="
for /f "tokens=2 delims==" %%A in ('
    wmic process where (ProcessId^=!NODEPID!^) get ParentProcessId /format:list
') do (
    set /a "PARENTPID=%%A"
)

if defined PARENTPID (
    rem === 強制結束全部行程 ===
    echo [3] kill all PID !PARENTPID! 
    taskkill /F /T /PID !PARENTPID!
    
    rem === 等 2 秒確定釋放完畢 ===
    echo [4] waiting 2 sec
    timeout /t 2 >nul
  
) else (
    echo   [ERROR] 無法取得 Parent PID
)

rem === 重新啟動 Node-RED ===
echo [5] start NodeRED
start "" "%APPDATA%\npm\node-red.cmd"

endlocal

```
