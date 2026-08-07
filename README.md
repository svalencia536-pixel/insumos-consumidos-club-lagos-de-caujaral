# Insumos Consumidos — Club Lagos de Caujaral

Cruza la venta facturada (Zeus/POS) contra el libro de recetas del club para
calcular cuánta materia prima real se consumió, a qué costo, y en qué centro.

Es una página **estática de un solo archivo**: todo el libro de recetas viene
embebido en `index.html` y el archivo de ventas se carga **en el navegador**
del usuario (arrastrar y soltar el `.xlsx`) — nada se sube a este servidor ni
se guarda en ningún lado. `server.js` solo sirve ese archivo, no procesa datos.

## Cómo funciona

```
Excel del club (Consulta1)  →  arrastrar .xlsx en el navegador  →  se lee y cruza ahí mismo
                                                                          ↑
                                                        libro de recetas ya embebido en index.html
```

- El cruce es por **código de fórmula** (columna `Codigo_inv` del archivo de
  ventas, la misma que usa Zeus internamente) — no por nombre, porque el POS y
  el libro de recetas usan numeraciones distintas.
- Si un producto no tiene fórmula pero sí código de inventario (una bebida que
  se vende tal cual, por ejemplo), se cuenta como su propio insumo directo.
- Los ingredientes que a su vez son subrecetas del libro se explotan de forma
  recursiva hasta llegar a la materia prima real.
- El `.xlsx` se lee en JavaScript puro dentro de la página (descomprime el ZIP
  con `DecompressionStream`, nativo del navegador, y parsea el XML con
  `DOMParser`) — no depende de ninguna librería externa.

No usa dependencias externas: solo módulos nativos de Node 18+.

## Rutas

| Ruta | Qué hace |
|---|---|
| `/` | La página |
| `/salud` | JSON con el estado, para el monitoreo de Railway |

## Desplegar en Railway

1. Subir este repo a GitHub (ya hecho).
2. En Railway: **New Project → Deploy from GitHub repo** y elegir este repo.
3. En **Settings → Networking**, generar el dominio público.

No hay variables de entorno obligatorias. Railway detecta Node por el
`package.json` y ejecuta `npm start`. No hay build.

## Mantenimiento

- **Para actualizar cifras de ventas**: no hay que tocar el repo ni volver a
  desplegar — cada usuario arrastra su propio `.xlsx` actualizado en el
  navegador cuando abre la página.
- **Si cambian recetas del libro de forma importante**, hay que reconstruir
  `index.html` con los datos nuevos de la hoja "Query 1" (y "BASE" para
  categorías) del libro de recetas, y volver a hacer `git push`. No es
  autoeditable desde la página.
