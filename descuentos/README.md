# Descuentos del Súper (CABA)

Dashboard semanal de promos bancarias en supermercados de Capital Federal,
filtrado por los bancos y tarjetas que tengas.

- `index.html`: la página lista para abrir (o servir con GitHub Pages).
- `app.html`: la misma página sin el esqueleto `<html>`/`<head>` (se usa para publicarla como Artifact).

## Cómo actualizar las promos
Los datos están en el bloque `<script id="promo-data" type="application/json">`.
Cada promo tiene: `ent` (banco/billetera), `sup` (súper), `dias` (1=lunes … 7=domingo),
`pct`, `tope` (`null` = sin tope, `"app"` = no informado, o `{m, p}` con monto y período),
`min` (compra mínima), `tj` (tarjetas requeridas), `conf` (alta/media/baja) y `fu` (fuentes).
Cambiá también `VIG` (vigencia) y `periodo` cuando cargues un mes nuevo.
Después de editar `app.html`, regenerá `index.html`:

```sh
cd descuentos
{ echo '<!DOCTYPE html><html lang="es"><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">'; sed -n '1,/<\/style>/p' app.html; echo '</head><body>'; sed '1,/<\/style>/d' app.html; echo '</body></html>'; } > index.html
```

## Fuentes oficiales (scraper)
`scraper/scrape.js` lee las promos que publica cada súper y las guarda normalizadas en
`oficial/oficial-AAAA-MM-DD.json` (porcentaje, días o fechas, tope, mínimo, vigencia, canal y texto legal):

| Súper | De dónde sale |
|---|---|
| Jumbo, Disco, Vea | `jumbo.com.ar/api/dataentities/JN/documents/bankDiscount` (JSON) |
| Carrefour | GraphQL `GetPromotions` de `carrefour.com.ar/descuentos-bancarios` |
| Día | bloque de promos embebido en el HTML de `/medios-de-pago-y-promociones` |
| Coto | legales de `coto.com.ar/terminos-descuentos` (con Playwright) |

```sh
NODE_PATH=$(npm root -g) node descuentos/scraper/scrape.js
```
Ese JSON no reemplaza los datos de la página: se usa para confirmar (`"conf":"oficial"`) o corregir
las promos que vienen de notas de prensa. ChangoMás todavía no tiene fuente oficial.
