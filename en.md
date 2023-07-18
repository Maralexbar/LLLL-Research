# English

## Introduction

In this document, I will explain how to extract assets such as textures, videos, audio, and Unity scripts easily, taking advantage of the lack of encryption in most game assets.

## Prerequisites

Before getting started, make sure you have the following:

- Access to the game assets, which are located at `/sdcard/Android/data/com.oddno.lovelive/files`.
- The game should be configured to download all files instead of using a light/sequential download. This can be modified in the game settings.

## Extracting the Assets

To extract the assets, we will need to use the `adb` (Android Debug Bridge) command. Follow the steps below:

1. Connect your Android device to the computer using a USB cable.

2. Open a terminal or command prompt and make sure you have `adb` installed and properly configured.

3. Execute the following command to make a copy of the game files to your desired destination folder:

```bash
adb pull /sdcard/Android/data/com.oddno.lovelive/files ruta-de-destino
```

Replace `destination-path` with the location of the folder where you want to save the extracted files.

Once this step is completed, you will have the game files in your destination folder.

## File Structure

Inside the destination folder, you will find the following file structure:
 
    ├── 📂files
     		    ├── 📂D (Downloaded Data)
     		    |		└── 📂 ...
    		    ├── 📂il2cpp (Unity Resources along with global-metadata.dat)
    		    ├── 📂M (Encrypted DB positions with a -journal)
    		    └── 📂Unity (Not relevant, only telemetry cache)
				


The "D" folder is the one we are interested in, as it contains the downloaded data. It is composed of many subfolders identified by two letters or digits, and each of these subfolders contains files with names that appear to be a hash (although they don't match the MD5 format).

The first two letters or digits of the filename correspond to the respective subfolder.

## Identifying Files

So far, I have identified four different types of files based on their header or magic number.

- Cri Video Files (USM) start with the header: `43 52 49 44` (`CRID`)
- Cri Audio Files (HCA) contained in AFS2 have the header: `41 46 53 32` (`AFS2`)
- CPK Files have the header: `40 55 54 46` (`@UTF`)
- UnityFS Files, but with 2 bytes in the header: `AB 00`, which need to be removed for proper usage.

## File Classification

In addition to the aforementioned file types, there are also unknown binary files that have not been identified yet.

### Classification Scripts

A series of PowerShell scripts have been developed to classify the files. These scripts add an extension based on the header found in each file. To execute the scripts, open PowerShell in the "D" folder, where the assets are located.

- For USM Video Files (.usm)
```powershell
  Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 4 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "43524944" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".usm") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```
- For AFS2 Files (.afs2)
```powershell
Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 4 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "41465332" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".afs2") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```
- For UnityFS Files (.unity)
```powershell
Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 3 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "AB0055" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".unity") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```
- For CPK Files (.utf8)
```powershell
Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 4 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "40555446" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".utf8") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```
### Removing UnityFS Header

If you want to remove the 2 bytes from the UnityFS Files (.unity), you can execute the following script. Please note that this will replace all previously renamed .unity files.

```powershell
Get-ChildItem -File -Recurse -Filter "*.unity" | ForEach-Object { $fs = New-Object System.IO.FileStream($_.FullName, [System.IO.FileMode]::Open); $fs.Seek(2, [System.IO.SeekOrigin]::Begin) > $null; $buffer = New-Object byte[] ($fs.Length - 2); $fs.Read($buffer, 0, $buffer.Length) > $null; $fs.Close(); $fsTemp = New-Object System.IO.FileStream([System.IO.Path]::GetTempFileName(), [System.IO.FileMode]::Create); $fsTemp.Write($buffer, 0, $buffer.Length); $fsTemp.Close(); Move-Item -Path $fsTemp.Name -Destination $_.FullName -Force; Write-Host "Archivo procesado: $($_.FullName)" }
```

## Opening the Files

- UnityFS Files (.unity) without the 2 bytes can be opened using [AssetStudioGUI](https://github.com/Perfare/AssetStudio).
- For AFS2 and USM Files, they should be extracted using [VGMToolbox](https://github.com/bxaimc/VGMToolbox) first. Then, the HCA audio files can be converted using [vgmstream](https://github.com/vgmstream/vgmstream) or by using a player that can read .hca files.
- Currently, no functional tool has been found to open CPK Files.

I hope this information is helpful for the classification and manipulation of the files.
