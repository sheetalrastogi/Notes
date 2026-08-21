```text
@echo off
title Selenium 4 Grid Startup

REM ==================================================
REM Grid Start Time  
REM ==================================================
echo Grid Started At : %DATE% %TIME% > gridStartTime.txt

REM ==================================================
REM Start Selenium Hub
REM ==================================================
start "Selenium Hub" cmd /k ^
java -jar selenium-server-4.35.0.jar hub

timeout /t 10 > nul

REM ==================================================
REM Start Chrome Node
REM ==================================================
start "Chrome Node 1" cmd /k ^
java -jar selenium-server-4.35.0.jar node ^
--hub http://localhost:4444

REM ==================================================
REM Start Firefox Node
REM ==================================================
start "Firefox Node 1" cmd /k ^
java -jar selenium-server-4.35.0.jar node ^
--hub http://localhost:4444

REM ==================================================
REM Start Edge Node
REM ==================================================
start "Edge Node 1" cmd /k ^
java -jar selenium-server-4.35.0.jar node ^
--hub http://localhost:4444

echo.
echo Selenium Grid Started Successfully
echo Start Time saved to gridStartTime.txt
pause

```
