# Español
Como algunos sabran el juego tiene una encriptacion nula en la mayoria de assets por lo cual es facil sacar algunas texturas, videos, audio y scripst de unity de forma sencilla.

Primero se necesitan los assets del juego que estan ubicados en `/sdcard/Android/data/com.oddno.lovelive/files` pero el juego tiene una descarga ligera/secuencial predeterminada por lo que hay que cambiar a descarga completa.

Una vez verificado que tengamos la descarga completa procedemos a hacer dump de los archivos, solo necesitaremos `adb` para eso.

`adb pull /sdcard/Android/data/com.oddno.lovelive/files carpeta-de-destino`

Una vez dentro de la carpeta de destino tenderemos la siguiente estructira de archivos

    ├── 📂files
     		    ├── 📂D (Datos descargados)
     		    |		└── 📂 ...
    		    ├── 📂il2cpp (Recursos de Unity junto al global-metadata.dat)
    		    ├── 📂M (Pos. BDs encriptada con un -journal)
    		    └── 📂Unity (Nada relevante solo cache de Telemetria)
				
				

Lo que realmente nos interesa es la carpeta D con los datos descargada, esta dividida en muchas subcarpetas compuestas por dos letras o digitos, dentro estan los archivos nombrados como si fuera un hash (parecen md5 pero al probar no coinciden), las dos primeras letras/digitos coinciden con la carpeta correspondiente.
