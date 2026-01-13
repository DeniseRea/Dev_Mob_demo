
# Documentación - Configuración de Firma Digital para Flutter

Esta documentación describe los pasos necesarios para configurar la firma digital del proyecto Flutter, requerida para la publicación en Google Play Store.

---

## 1. Generar la Clave de Firma (Keystore)

### Paso 1.1: Ejecutar el comando keytool

Abre tu terminal/CLI y ejecuta el siguiente comando para generar un archivo keystore:

```bash
keytool -genkey -v -keystore D:/Dev_Mob_demo/key/mi_app_keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias demokey1
```

> **Nota:** Este comando creará automáticamente el directorio `key/` en la raíz del proyecto si no existe.

### Paso 1.2: Ingresar datos solicitados

El comando anterior solicitará que ingreses la siguiente información:

| Campo | Ejemplo | Descripción |
|-------|---------|-------------|
| **Contraseña (Password)** | La que elijas | Contraseña del almacén de claves. Recuerda esta contraseña. |
| **Alias** | `demokey1` | Identificador único de la clave dentro del almacén. |
| **Otros datos** | Tu nombre, empresa, etc. | Información personal solicitada por keytool. |

---

## 2. Crear el archivo `key.properties`

### Paso 2.1: Ubicación del archivo

Crea un archivo llamado `key.properties` en la carpeta `android/` del proyecto:

```
android/key.properties
```

### Paso 2.2: Configurar el contenido

Rellena el archivo con los siguientes parámetros:

```properties
storePassword=<contraseña-del-paso-anterior>
keyPassword=<contraseña-del-paso-anterior>
keyAlias=demokey1
storeFile=D:\\Dev_Mob_demo\\key\\mi_app_keystore.jks
```

**Ejemplo para Windows:**
```properties
storePassword=miContraseña123
keyPassword=miContraseña123
keyAlias=demokey1
storeFile=D:\\Dev_Mob_demo\\key\\mi_app_keystore.jks
```

> ⚠️ **Importante:** En Windows, usa barras invertidas dobles (`\\`) en las rutas. Nunca compartas este archivo ni lo subas a control de versiones.

---

## 3. Configurar el archivo Gradle

### Paso 3.1: Editar `android/app/build.gradle.kts`

Navega al archivo de configuración de Gradle:

```
android/app/build.gradle.kts
```

### Paso 3.2: Agregar configuración de firma

Añade la siguiente configuración en el bloque `android`:

```kotlin
// ...existing code...

android {
    // ...existing code...
    
    signingConfigs {
        create("release") {
            val keystoreFile = rootProject.file("key.properties")
            val keystoreProperties = Properties()
            keystoreProperties.load(keystoreFile.inputStream())
            
            keyAlias = keystoreProperties["keyAlias"] as String
            keyPassword = keystoreProperties["keyPassword"] as String
            storeFile = file(keystoreProperties["storeFile"] as String)
            storePassword = keystoreProperties["storePassword"] as String
        }
    }
    
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            // ...existing code...
        }
    }
}
```

### Paso 3.3: Versión de Gradle

Si utilizas versiones antiguas de Gradle, es posible que recibas solicitudes de actualización. Se recomienda mantener la versión actualizada a la última versión estable.

### Paso 3.4: Limpiar y actualizar dependencias

Después de configurar los archivos anteriores, ejecuta los siguientes comandos en tu terminal:

```bash
flutter clean
flutter pub get
```

Estos comandos limpiarán la compilación anterior y descargarán las dependencias necesarias.

---

## 4. Verificar el AndroidManifest.xml

### Paso 4.1: Revisar permisos

Abre el archivo `android/app/src/main/AndroidManifest.xml` y verifica que contenga los permisos necesarios:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:label="[nombre-de-tu-proyecto]"
        ... >
    </application>
    
    <!-- Permisos requeridos -->
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

> **Nota:** El permiso `INTERNET` es esencial para la mayoría de aplicaciones. Agrega otros permisos según las características de tu app.

---

## 5. Verificar la Configuración de Gradle

### Paso 5.1: Revisar build.gradle.kts

Para verificar que la configuración de Android es correcta, revisa el bloque `android` en el archivo de compilación:

```
Ubicación: android/app/build.gradle.kts
```

Este archivo contiene todas las propiedades de configuración de tu build de Android. Puedes modificar valores como:

- `compileSdk` - Versión del SDK compilada
- `minSdk` - Versión mínima de Android soportada
- `targetSdk` - Versión objetivo de Android
- `versionCode` y `versionName` - Números de versión de tu app

---

## Checklist de Verificación

- [ ] Archivo `key.properties` creado en `android/`
- [ ] Archivo `mi_app_keystore.jks` existe en `android/key/`
- [ ] Contraseñas ingresadas correctamente en `key.properties`
- [ ] Configuración de firma agregada a `build.gradle.kts`
- [ ] `key.properties` agregado a `.gitignore` (por seguridad)
- [ ] Comandos `flutter clean` y `flutter pub get` ejecutados
- [ ] Permisos verificados en `AndroidManifest.xml`
- [ ] Configuración de Gradle revisada y validada
