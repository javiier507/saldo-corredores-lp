# AGENTS.md

Landing estática de Saldo Corredores: `index.html` y `privacy.html`, con CSS inline y sin build. Se publica en Netlify y la app la muestra en "Acerca de" dentro de un WebView.

## La landing hereda el estilo de la app

- En cada carga, la app inyecta estas variables con `!important` (`COLORES_DEL_TEMA` en `saldo-corredores-auto/mobile/.../ui/pantallas/AcercaScreen.kt`): `--primary-color`, `--on-primary-color`, `--surface-color`, `--on-surface-color`, `--on-surface-variant-color`, `--outline-color`.
- Todo color sale de esas variables, o de combinarlas con `color-mix`. No uses colores fijos fuera de los valores por defecto de `:root` y del bloque `prefers-color-scheme: dark`, que solo se ven fuera de la app.
- Una página nueva copia esos dos bloques de `index.html`.
- Si añades o renombras una variable, cámbiala también en la app.

## Navegación

Usa enlaces relativos (`index.html`, `privacy.html`). Los enlaces se abren dentro del WebView, que no tiene botón de volver, así que cada página secundaria lleva un enlace a `index.html`.
