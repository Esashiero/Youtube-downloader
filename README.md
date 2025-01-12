# YouTube Download Button Integration

Enhance your YouTube experience by adding a **Download Video** button using a Tampermonkey script, a custom URL handler, and a PowerShell script.

## Features

- **Download Button**: Adds a "Download Video" button to YouTube pages.
- **Automated Downloads**: Downloads videos directly using `yt-dlp`.
- **Background Execution**: Runs without displaying a console window.
- **Notifications**: Displays alerts on the YouTube page when downloads start and finish.
- **Auto-Open Folder**: Opens the download folder automatically after completion.

## Prerequisites

Ensure the following tools are installed:

- [yt-dlp](https://github.com/yt-dlp/yt-dlp)
- [Tampermonkey](https://www.tampermonkey.net/) browser extension
- [PS2EXE](https://github.com/MScholtes/PS2EXE) for converting PowerShell scripts to executables

## Setup Guide

### 1. Create the PowerShell Script

1. Open a text editor (e.g., Notepad or VS Code).
2. Paste the following code:

   ```powershell
   # youtube-dl-handler.ps1

   param(
       [Parameter(Mandatory=$true)]
       [string]$url
   )

   # Set up logging
   $logFile = "C:\Scripts\ytdl-log.txt"
   $downloadDir = "D:\CORE\Vidéos\Vidéos - DL"

   function Write-Log {
       param($Message)
       $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
       "$timestamp - $Message" | Add-Content $logFile
   }

   try {
       if (!(Test-Path $downloadDir)) {
           New-Item -ItemType Directory -Path $downloadDir -Force | Out-Null
           Write-Log "Created directory: $downloadDir"
       }

       $youtubeUrl = $url -replace "ytdl://", ""
       $youtubeUrl = [uri]::UnescapeDataString($youtubeUrl)
       Write-Log "Starting download for: $youtubeUrl"

       Set-Location $downloadDir

       $startInfo = New-Object System.Diagnostics.ProcessStartInfo
       $startInfo.FileName = "yt-dlp"
       $startInfo.Arguments = "--cookies-from-browser firefox `"$youtubeUrl`""
       $startInfo.RedirectStandardOutput = $true
       $startInfo.RedirectStandardError = $true
       $startInfo.UseShellExecute = $false
       $startInfo.CreateNoWindow = $true
       $startInfo.WindowStyle = 'Hidden'
       $startInfo.WorkingDirectory = $downloadDir

       $process = New-Object System.Diagnostics.Process
       $process.StartInfo = $startInfo
       $process.Start() | Out-Null

       $stdOutput = $process.StandardOutput.ReadToEnd()
       $stdError = $process.StandardError.ReadToEnd()
       $process.WaitForExit()

       if ($process.ExitCode -eq 0) {
           Write-Log "Download completed successfully"
           Start-Process "explorer.exe" $downloadDir
       } else {
           Write-Log "Download failed with exit code: $($process.ExitCode). Error output: $stdError"
       }
   } catch {
       Write-Log "Script error: $($_.Exception.Message)"
   } finally {
       if ($null -ne $process) {
           $process.Dispose()
       }
   }
