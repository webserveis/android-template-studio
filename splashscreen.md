

# Intro

Plantillas de android para simplificar la creación de aplicaciones, variedad de estilo de pantallas y patrones de diseño. Las vistas se han maqueteado siguiendo la guia de estilo de Google y la funcionalidad mediante lenguaje Kotlin (100% nativas)

## Base de proyecto
- base principal de todos los módulos, Colores, Modo oscuro automático.

## Splash screens
 - [Splash screen](https://github.com/webserveis/android-template-studio/blob/main/splashscreen.md)
 - Splash screen con progressView
 
 ## Welcome screens
 - Welcome screen simple
	 pantalla de inicio logotipo y marca comercial.
 - Welcome screen multiple summary
	 pantalla de incio con multiples sumarios.
 - Welcome screen feature apps (Whats the new)
	 pantalla de inicio con resaltar caracteristicas de la app.

## Main screens
- Main screen Hello world
	 pantalla de contenido principal vacia
- Main screen Top panel
	pantalla de contenido con un panel que se oculta al realizar scroll.
- Main screen with tabs
	pantalla de contenido agrupado por pestañas
	
## List screens
- List screen simple
	lista simple de solo lectura
- List screen multi-view (headers)
	lista con multiple vistas, encabezados
- List combined diferent group list views (Nested RecyclerView)
	lista vertical combinada de diferente tipos de vistas, desplazamiento horizontal
- List CRUD operations in memory (Swipe View for deletion)
	lista dinánimca con operaciones Create Read Update Delete

## Detail screens
- Read detail screen
	Vista de detalle solo lectura
- CRUD detail screen
	Vista de detalle con operación de Create Read Update Delete

## Settings screens
- Setting screen with darkmode
	Pantalla de ajustes con selector de modo oscuro

## Gallery screens
 - List images
	 Pantalla para mostrar una galeria
 - Image Viewer
	 Visor de imágenes

## Map screen
- Map screen with google maps
	Pantalla de carga de maps

## Dialogs and Pickers
- Alert dialog
	Cuadro de diálogo de alerta
- Radio list dialog
	Cuadro de diálogo con selector de elementos
- Check list dialog
	Cuadro de diálogo con selector de elementos multiple
- Date Picker
	Para poder escoger una fecha
- Time picker
	Para poder escoger una hora
- Date Time picker
	Para poder escoger fecha y hora
- Color picker
	Para poder escoger un color predeterminado
- Days week picker
	Para poder selecionar multiples dias de la semana

## Bottom Sheet pickers
- Bottom sheet custom
	Panel inferior deslizante
- Bottom sheet 3 states (full screen)
	Panel inferior deslizante de 3 estados
- Action bottom sheet
	Panel inferior con menú de acciones

## Menus and Navigation
- Menu simple
	Carga de menú principal o secundario
- Side menu navigation
	Implementar menú lateral de navegación
- Bottom navigation
	Implementar barra de navegación inferior

## State screens
- Loading, Empty, Error screen
	Mostrar pantallas de carga, vacio o de error en la obtención de datos
- Event app screen (Full screen)
	Mostrar pantalla de evento en pantalla completa

## Data Sources
 - Combine localSource and RemoteSource
	 Combinar obtención de datos de remoto y local
- Check if need update data
	Checkear si se requiere actualizar datos de una fuente a otra

### LocalSource
- Load data from static source (assets)
	Obtener datos staticos de assets
- Load data from database
	Obtener datos de base de datos
- CRUD database
	Obtener y operar sobre los datos de la base de datos

### RemoteSource
- Load data form remote source (API json)
	Obtener datos remotadamente, API rest

## LiveData observables
- observate internet conection
	Monitorizar el estado de la conexión de internet
- observate GPS location
	Monitorizar la posición del dispositivo

## Services
- Launch service
	Lanzar un servicio
- Launch and comunicate service
	Lanzar un servicio y comunicarse con el
- Mediaplayer service
	Lazar un servicio multimedia

## Lock Screen
- Lock screen with pin code
	Pantalla de bloqueo con validación de pin
- Lock screen with pattern and biometric system
	Pantalla de bloqueo con patrón y sistema biométrico
