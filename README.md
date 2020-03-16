# AndResGuard

[![Estado de compilación](https://travis-ci.org/shwenzhang/AndResGuard.svg?branch=master)](https://travis-ci.org/shwenzhang/AndResGuard)
[ ![Descargar](https://api.bintray.com/packages/wemobiledev/maven/com.tencent.mm%3AAndResGuard-core/images/download.svg) ](https://bintray.com/wemobiledev/maven/com.tencent.mm%3AAndResGuard-core/_latestVersion)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-AndResGuard-green.svg?style=true)](https://android-arsenal.com/details/1/3034)

*Lee esto en otros idiomas: [Español](README.md), [简体中文](README.zh-cn.md).*

`AndResGuard` es una herramienta para reducir el tamaño de su apk, funciona como el` ProGuard` para el código fuente de Java, pero solo apunta a los archivos de recursos. Cambia `res/drawable/wechat` a` r/d/a`, y cambia el nombre del archivo de recursos `wechat.png` a` a.png`. Finalmente, vuelve a empaquetar el apk con 7zip, lo que obviamente puede reducir el tamaño del paquete.

`AndResGuard` es rápido, y ** NO ** necesita los códigos fuente. Ingrese un apk de Android, luego podemos obtener un apk de 'ResGuard' en unos segundos.

Algunos usos de `AndResGuard` son:

1. Ofuscar los recursos de Android. Contiene todo el tipo de recurso (como dibujable,diseño,cadena ...). Puede evitar que tu apk sea revertido por `Apktool`.

2. Reducir el tamaño del apk. Puede reducir el "resources.arsc" y el tamaño del paquete obviamente.

3. Reempaquetado con `7zip`. Es compatible con apk de reempaquetado con `7zip`, y podemos especificar el método de compresión para cada archivo.

`AndResGuard` es una herramienta de línea de comandos, es compatible con Windows, Linux y Mac. Le sugerimos que use 7zip en la plataforma Linux o Mac para obtener una mayor relación de compresión.

## Cómo utilizar
### Con Gradle
Esto ha sido lanzado el `Bintray`
```gradle
apply plugin: 'AndResGuard'

buildscript {
    repositories {
        jcenter()
        google()
    }
    dependencies {
        classpath 'com.tencent.mm:AndResGuard-gradle-plugin:1.2.17'
    }
}

andResGuard {
    // mappingFile = file("./resource_mapping.txt")
    mappingFile = null
    use7zip = true
    useSign = true
    // Mantendrá la ruta de origen de sus recursos cuando sea cierto
    keepRoot = false
    // Si se establece, la columna de nombre en arsc que se necesita proteger se mantendrá en este valor
    fixedResName = "arg"
    // Fusionará los recursos duplicados, pero no confíe demasiado en esta característica.
    // siempre es mejor eliminar recursos duplicados del repositorio
    mergeDuplicatedRes = true
    whiteList = [
        // your icon
        "R.drawable.icon",
        // for fabric
        "R.string.com.crashlytics.*",
        // for google-services
        "R.string.google_app_id",
        "R.string.gcm_defaultSenderId",
        "R.string.default_web_client_id",
        "R.string.ga_trackingId",
        "R.string.firebase_database_url",
        "R.string.google_api_key",
        "R.string.google_crash_reporting_api_key"
    ]
    compressFilePattern = [
        "*.png",
        "*.jpg",
        "*.jpeg",
        "*.gif",
    ]
    sevenzip {
        artifact = 'com.tencent.mm:SevenZip:1.2.17'
        //path = "/usr/local/bin/7za"
    }

    /**
    * Opcional: si finalApkBackupPath es nulo, AndResGuard sobrescribirá el apk final
    * a la ruta que ensamblan [Tarea] escribir en
    **/
    // finalApkBackupPath = "${project.rootDir}/final.apk"
}
```

### Comodín
El comodín de soporte 'whiteList' y 'compressFilePattern' incluye ? * +.

```
?	Cero o un personaje
*	Cero o más de carácter
+	Uno o más de personaje
```

### WhiteList
Necesita poner todos los recursos que acceden a través de `getIdentifier` en whiteList.
**Puede encontrar más configuraciones 'whitsList' de SDK de terceros en [white_list.md] (doc/white_list.md). Bienvenido PR sus configuraciones que no están incluidas en white_list.md**

WhiteList solo funciona en specsName de recursos, no mantendría la ruta del recurso.
Si desea mantener la ruta, utilice `mappingFile` para implementarla.

Por ejemplo, queremos mantener la ruta del ícono, necesitamos agregarlo a continuación en nuestro `mappingFile`.
```
mapeo de ruta res:
    res/mipmap-hdpi-v4 -> res/mipmap-hdpi-v4
    res/mipmap-mdpi-v4 -> res/mipmap-mdpi-v4
    res/mipmap-xhdpi-v4 -> res/mipmap-xhdpi-v4
    res/mipmap-xxhdpi-v4 -> res/mipmap-xxhdpi-v4
    res/mipmap-xxxhdpi-v4 -> res/mipmap-xxxhdpi-v4
```

### Cómo lanzar
Si está utilizando `Android Studio`, puede encontrar la opción de generar tarea en el grupo ```andresguard```.
O, alternativamente, ejecuta ```./gradlew resguard [BuildType | Sabor]``` en su terminal. El formato del nombre de la tarea es el mismo que `ensamblar`.

### Sevenzip
El `sevenzip` en el archivo gradle se puede establecer por `path` o `artifact`. Se permiten múltiples tareas, pero el ganador es ** always** `path`.

### Result
Si finalApkBackupPath es nulo, AndResGuard sobrescribirá el APK final en la ruta que ensambla la escritura [Tarea]. De lo contrario, se almacenará en la ruta que asignó.

### Other
[Buscando más detalles](doc/how_to_work.md)


## Problema conocido
1. El primer elemento de la lista que devuelve `AssetManager#list(String path)` es una cadena vacía cuando usa el APK comprimido por 7zip. [#162](https://github.com/shwenzhang/AndResGuard/issues/162)

## Best Practise
1. ** NO ** agregue `resources.arsc` en `compressFilePattern` a menos que el tamaño de la aplicación sea realmente importante para usted.([#84](https://github.com/shwenzhang/AndResGuard/issues/84) [#233](https://github.com/shwenzhang/AndResGuard/issues/233))
2. ** NO ** habilite la compresión 7zip (`use7zip`) cuando distribuya su APLICACIÓN en Google Play. Evitará el parche de archivo por archivo al actualizar su aplicación. ([#233](https://github.com/shwenzhang/AndResGuard/issues/233))


## Gracias
[Apktool](https://github.com/iBotPeaches/Apktool) Connor Tumbleson

[v2sig](https://github.com/shwenzhang/AndResGuard/pull/133) @jonyChina162
