# Splash screen

Módulo para crear pantalla de bienvenida (Splash Screen)

## 1 Creando los assets

**Dimensiones de la pantalla de inicio**
 - LDPI:
    - Portrait: 200x320px
    - Landscape: 320x200px
 - MDPI:
    - Portrait: 320x480px
    - Landscape: 480x320px
 - HDPI:
    - Portrait: 480x800px
    - Landscape: 800x480px
 - XHDPI:
    - Portrait: 720px1280px
    - Landscape: 1280x720px
 - XXHDPI
    - Portrait: 960x1600px
    - Landscape: 1600x960px
 - XXXHDPI 
    - Portrait: 1280x1920px
    - Landscape: 1920x1280px

> Lo más recomendable es usar los assets de XXHDPI es decir 960x1600px

Diseñamos la pantalla de inicio siguiendo la siguiente estructura


 - El color de fondo, se puede heredar del sistema si el modo oscuro está activo o no
 - para el logotipo splash_screen_cover.png
 - para la marca splash_screen_branding.png

los asignamos a la carpeta `drawable-xxhdpi`

Ahora solo falta unir los assets lo haremos con un layer dentro de un drawable

```xml
<?xml version="1.0" encoding="utf-8"?>  
<layer-list xmlns:android="http://schemas.android.com/apk/res/android"  
  android:opacity="opaque">  
  <!-- splash screen background color -->  
  <item android:drawable="@android:color/background_light" />  
  
  <!-- splash screen cover -->  
  <item  
  android:drawable="@drawable/splash_screen_cover"  
  android:gravity="center" />  
  
  <!-- splash screen branding -->  
  <item  
  android:bottom="72dp">  
 <bitmap  
  android:alpha="0.66"  
  android:gravity="bottom"  
  android:src="@drawable/splash_screen_branding" />  
 </item>  
</layer-list>
```

## 2 Definiendo el tema
En `styles.xml` crearemos un nuevo tema llamado `SplashTheme` y aquí con la propiedad `android:windwoBackground` le asignamos el asset de la splashscreen.

```xml
<!-- Splash Screen theme. -->  
<style name="SplashTheme" parent="Theme.MaterialComponents.NoActionBar">  
  <item name="android:windowBackground">@drawable/splash_background</item>  
 <item name="android:statusBarColor">@android:color/background_light</item>  
 <item name="android:navigationBarColor">@android:color/background_light</item>  
  <item name="android:windowLightStatusBar">true</item>  
</style>
 ``` 

## 3 Creando la vista splash screen
Crear una nueva actividad y llamarla SplashScreenActivity

```kotlin
class SplashScreenActivity : AppCompatActivity() {  
 
  override fun onCreate(savedInstanceState: Bundle?) {  
  super.onCreate(savedInstanceState)  
  
  loadSplashScreen()  
  
 }  
  private fun loadSplashScreen() {  
  val delayTime = resources.getInteger(android.R.integer.config_longAnimTime).toLong() * 2  
  Handler(Looper.getMainLooper()).postDelayed({ launchMainScreen() }, delayTime)  
 }  
  private fun launchMainScreen() {  
  val intent = Intent(this, MainActivity::class.java)  
  startActivity(intent)  
  overridePendingTransition(android.R.anim.fade_in, android.R.anim.fade_out)  
  this.finish()  
 }  
}
```

## 4 Asignar la splash screen como principal
Asignar la splashscreen que sea la actividad principal de la aplicación en `AndroidManifest.xml`
```xml
<activity  
  android:name=".SplashScreenActivity"  
  android:label="@string/app_name"  
  android:launchMode="singleTop"  
  android:exported="true"  
  android:screenOrientation="portrait"  
  android:theme="@style/SplashTheme">  
 <intent-filter>  
 <action android:name="android.intent.action.MAIN" />  
 <category android:name="android.intent.category.LAUNCHER" />  
 </intent-filter>  
</activity>
```
