# Landing Page - Saldo Corredores

Esta landing page está diseñada para ser incrustada en una **WebView de Android** y adaptarse dinámicamente a los colores de Material Design 3 de tu aplicación.

## Características

- **CSS Variables dinámicas**: Los colores pueden ser inyectados desde Kotlin
- **Material Design 3 compatible**: Usa la nomenclatura de Material Design 3
- **Responsive**: Se adapta a diferentes tamaños de pantalla
- **Sin dependencias**: HTML/CSS puro, sin frameworks

## Variables CSS Disponibles

La landing page usa las siguientes CSS Variables que mapean directamente a Material Design 3:

```css
--primary-color              /* MaterialTheme.colorScheme.primary */
--on-primary-color           /* MaterialTheme.colorScheme.onPrimary */
--surface-color              /* MaterialTheme.colorScheme.surface */
--on-surface-color           /* MaterialTheme.colorScheme.onSurface */
--on-surface-variant-color   /* MaterialTheme.colorScheme.onSurfaceVariant */
--outline-color              /* MaterialTheme.colorScheme.outline */
```

## Integración con Android/Kotlin

### Opción 1: Reemplazar AcercaScreen con WebView

Modifica `AcercaScreen.kt` para usar WebView en lugar de código estático:

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun AcercaScreen(onNavigateBack: () -> Unit) {
    val colorScheme = MaterialTheme.colorScheme

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Acerca de") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, "Volver")
                    }
                },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = colorScheme.primary,
                    titleContentColor = Color.White,
                    navigationIconContentColor = Color.White
                )
            )
        }
    ) { paddingValues ->
        AndroidView(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues),
            factory = { context ->
                WebView(context).apply {
                    settings.javaScriptEnabled = true
                    settings.domStorageEnabled = true

                    // Cargar la landing page
                    loadUrl("file:///android_asset/landing.html")

                    // Inyectar colores después de cargar la página
                    webViewClient = object : WebViewClient() {
                        override fun onPageFinished(view: WebView?, url: String?) {
                            super.onPageFinished(view, url)

                            // Convertir colores de Compose a formato hexadecimal
                            val primary = colorScheme.primary.toHexColor()
                            val onPrimary = colorScheme.onPrimary.toHexColor()
                            val surface = colorScheme.surface.toHexColor()
                            val onSurface = colorScheme.onSurface.toHexColor()
                            val onSurfaceVariant = colorScheme.onSurfaceVariant.toHexColor()
                            val outline = colorScheme.outline.toHexColor()

                            // Inyectar variables CSS
                            view?.evaluateJavascript("""
                                document.documentElement.style.setProperty('--primary-color', '$primary');
                                document.documentElement.style.setProperty('--on-primary-color', '$onPrimary');
                                document.documentElement.style.setProperty('--surface-color', '$surface');
                                document.documentElement.style.setProperty('--on-surface-color', '$onSurface');
                                document.documentElement.style.setProperty('--on-surface-variant-color', '$onSurfaceVariant');
                                document.documentElement.style.setProperty('--outline-color', '$outline');
                            """.trimIndent(), null)
                        }
                    }
                }
            }
        )
    }
}

// Función de extensión para convertir Color de Compose a Hex
fun Color.toHexColor(): String {
    val red = (this.red * 255).toInt()
    val green = (this.green * 255).toInt()
    val blue = (this.blue * 255).toInt()
    return String.format("#%02X%02X%02X", red, green, blue)
}
```

### Opción 2: WebView standalone (sin TopAppBar)

Si prefieres que la WebView ocupe toda la pantalla sin TopAppBar:

```kotlin
@Composable
fun AcercaScreen(onNavigateBack: () -> Unit) {
    val colorScheme = MaterialTheme.colorScheme

    BackHandler { onNavigateBack() }

    AndroidView(
        modifier = Modifier.fillMaxSize(),
        factory = { context ->
            WebView(context).apply {
                settings.javaScriptEnabled = true
                settings.domStorageEnabled = true

                loadUrl("file:///android_asset/landing.html")

                webViewClient = object : WebViewClient() {
                    override fun onPageFinished(view: WebView?, url: String?) {
                        super.onPageFinished(view, url)

                        val primary = colorScheme.primary.toHexColor()
                        val onPrimary = colorScheme.onPrimary.toHexColor()
                        val surface = colorScheme.surface.toHexColor()
                        val onSurface = colorScheme.onSurface.toHexColor()
                        val onSurfaceVariant = colorScheme.onSurfaceVariant.toHexColor()
                        val outline = colorScheme.outline.toHexColor()

                        view?.evaluateJavascript("""
                            document.documentElement.style.setProperty('--primary-color', '$primary');
                            document.documentElement.style.setProperty('--on-primary-color', '$onPrimary');
                            document.documentElement.style.setProperty('--surface-color', '$surface');
                            document.documentElement.style.setProperty('--on-surface-color', '$onSurface');
                            document.documentElement.style.setProperty('--on-surface-variant-color', '$onSurfaceVariant');
                            document.documentElement.style.setProperty('--outline-color', '$outline');
                        """.trimIndent(), null)
                    }
                }
            }
        }
    )
}
```

## Cómo agregar el archivo HTML a tu proyecto Android

1. Crea la carpeta `assets` en tu proyecto:
   ```
   app/src/main/assets/
   ```

2. Copia `index.html` a esa carpeta y renómbralo a `landing.html`:
   ```
   app/src/main/assets/landing.html
   ```

3. El archivo será accesible desde WebView con:
   ```kotlin
   loadUrl("file:///android_asset/landing.html")
   ```

## Dependencias necesarias

Agrega estas importaciones a tu archivo `AcercaScreen.kt`:

```kotlin
import android.webkit.WebView
import android.webkit.WebViewClient
import androidx.activity.compose.BackHandler
import androidx.compose.ui.viewinterop.AndroidView
import androidx.compose.ui.graphics.Color
```

## Permisos (si cargas desde URL externa)

Si decides hospedar la landing page en un servidor web, necesitarás agregar permisos de Internet en `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

**Nota**: No es necesario si cargas desde `assets` locales.

## Testing

Para probar que los colores se inyectan correctamente:

1. Cambia entre tema claro/oscuro en tu dispositivo Android
2. La WebView debería actualizar sus colores automáticamente
3. Verifica que todos los elementos usan los colores correctos del `MaterialTheme.colorScheme`

## Soporte

- **Tema claro/oscuro**: La landing page se adapta automáticamente al tema de la app
- **Material Design 3**: Usa la paleta de colores oficial de Material You
- **Responsive**: Se adapta a tablets y teléfonos
