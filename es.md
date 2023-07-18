# Español

## Introducción

En este documento, explicaré cómo extraer assets como texturas, videos, audio y scripts de Unity de forma sencilla, aprovechando la falta de encriptación en la mayoría de los assets del juego.

## Requisitos previos

Antes de comenzar, asegúrate de tener los siguientes elementos:

- Acceso a los assets del juego, los cuales se encuentran en la ruta `/sdcard/Android/data/com.oddno.lovelive/files`.
- El juego debe estar configurado para descargar todos los archivos en lugar de una descarga ligera/secuencial. Esto se puede modificar en la configuración del juego.

## Extracción de los assets

Para extraer los assets, necesitaremos utilizar el comando `adb` (Android Debug Bridge). Sigue los pasos a continuación:

1. Conecta tu dispositivo Android al ordenador mediante un cable USB.

2. Abre una terminal o línea de comandos y asegúrate de tener `adb` instalado y configurado correctamente.

3. Ejecuta el siguiente comando para realizar una copia de los archivos del juego en tu carpeta de destino:

```bash
adb pull /sdcard/Android/data/com.oddno.lovelive/files ruta-de-destino
```
Reemplaza ruta-de-destino con la ubicación de la carpeta donde deseas guardar los archivos extraídos.

Una vez completado este paso, tendrás los archivos del juego en tu carpeta de destino.

## Estructura de archivos

Dentro de la carpeta de destino, encontrarás la siguiente estructura de archivos:

    ├── 📂files
     		    ├── 📂D (Datos descargados)
     		    |		└── 📂 ...
    		    ├── 📂il2cpp (Recursos de Unity junto al global-metadata.dat)
    		    ├── 📂M (Pos. BDs encriptada con un -journal)
    		    └── 📂Unity (Nada relevante solo cache de Telemetria)
				
				

La carpeta "D" es la que nos interesa, ya que contiene los datos descargados. Está compuesta por muchas subcarpetas identificadas por dos letras o dígitos, y cada una de estas subcarpetas contiene archivos con nombres que parecen ser un hash (aunque no coinciden con el formato MD5).

Las dos primeras letras o dígitos del nombre de cada archivo coinciden con la subcarpeta correspondiente.

## Identificar archivos

Hasta el momento he encontrado 4 tipos diferentes de archivos gracias al encabezado o número mágico.

- Archivos de Video Cri (USM) empiezan con el encabezado: `43 52 49 44` (`CRID`)
- Archivos de Audio Cri (HCA) contenidos en AFS2, encabezado: `41 46 53 02` (`AFS2`)
- Archivos CPK con encabezado: `40 55 54 46` (`@UTF`)
- Archivos UnityFS, pero con 2 bytes en el encabezado: `AB 00`, los cuales deben ser removidos para su uso correcto.

## Clasificación de archivos

Además de los tipos de archivos mencionados anteriormente, también hay archivos binarios desconocidos que no han sido identificados hasta el momento.

### Scripts de clasificación

Se han desarrollado una serie de scripts en PowerShell para clasificar los archivos. Estos scripts agregan una extensión dependiendo del encabezado que se encuentre en cada archivo. Para ejecutar los scripts, abre PowerShell en la carpeta "D", donde se encuentran los assets.

- Para archivos de Video USM (.usm)
```powershell
  Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 4 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "43524944" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".usm") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```
- Para archivos AFS2 (.afs2)
```powershell
Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 4 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "41465302" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".afs2") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```
- Para archivos de UnityFS (.unity)
```powershell
Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 3 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "AB0055" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".unity") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}
}
```
- Para archivos CPK (.utf8)
```powershell
Get-ChildItem -File -Recurse  |  ForEach-Object {if (-join(Get-Content -Encoding Byte -TotalCount 4 -Path $_.FullName | ForEach-Object { $_.ToString("X2") }) -eq "40555446" ) {Rename-Item -Path $_.FullName -NewName ($_.BaseName + ".utf8") -Force ; Write-Host "Archivo procesado: $($_.FullName)"}}
```

### Remover encabezado de UnityFS

Si deseas remover los 2 bytes de los archivos (.unity), puedes ejecutar el siguiente script. Ten en cuenta que esto reemplazará todos los archivos .unity previamente renombrados.

```powershell
Get-ChildItem -File -Recurse -Filter "*.unity" | ForEach-Object { $fs = New-Object System.IO.FileStream($_.FullName, [System.IO.FileMode]::Open); $fs.Seek(2, [System.IO.SeekOrigin]::Begin) > $null; $buffer = New-Object byte[] ($fs.Length - 2); $fs.Read($buffer, 0, $buffer.Length) > $null; $fs.Close(); $fsTemp = New-Object System.IO.FileStream([System.IO.Path]::GetTempFileName(), [System.IO.FileMode]::Create); $fsTemp.Write($buffer, 0, $buffer.Length); $fsTemp.Close(); Move-Item -Path $fsTemp.Name -Destination $_.FullName -Force; Write-Host "Archivo procesado: $($_.FullName)" }
```


## Abrir los archivos

- Los UnityFS (.unity) sin los 2 bytes se pueden abrir con [AssetStudioGUI](https://github.com/Perfare/AssetStudio "AssetStudioGUI").
- Para los archivos AFS2 y USM, primero deben ser extraídos con [VGMToolbox](https://github.com/bxaimc/VGMToolbox "VGMToolbox"). Luego, los archivos de audio HCA se pueden convertir con [vgmstream](https://github.com/vgmstream/vgmstream "vgmstream") o utilizando un reproductor que pueda leer archivos .hca.
- Por el momento, no se ha encontrado una herramienta que funcione para abrir los archivos .cpk.

Espero que esta información sea útil para la clasificación y manipulación de los archivos.

