# Puentes

Becas, movilidades y convocatorias académicas para la comunidad Udelar, reunidas en un solo lugar.

Sitio: https://puentesuy.github.io/puentes/

Puentes es un proyecto independiente. No es un sitio oficial de la Universidad de la República.

## Archivos

| Archivo | Qué contiene | Cuándo se toca |
|---|---|---|
| `index.html` | Estructura y textos fijos de la página | Casi nunca |
| `estilos.css` | Colores, tipografías y diseño | Al mejorar la estética |
| `app.js` | Lógica: filtros, días que faltan, línea de cierres. Arriba de todo está `CONFIG` | Al agregar funciones o completar `CONFIG` |
| `convocatorias.json` | Los datos de las convocatorias | Cada semana |

## Actualizar las convocatorias

Todo se hace en `convocatorias.json`:

1. Cambiá `"actualizado"` por la fecha del relevamiento (formato `AAAA-MM-DD`).
2. Para agregar una convocatoria, copiá una entrada completa (desde `{` hasta `}`), pegala en la lista y cambiá sus datos.
3. Las convocatorias vencidas desaparecen solas del sitio. Podés borrarlas cuando quieras.

Campos de cada convocatoria:

| Campo | Obligatorio | Ejemplo o valores posibles |
|---|---|---|
| `titulo` | Sí | `"DAAD · Estancias de investigación en Alemania"` |
| `corto` | No | Nombre breve para la línea de cierres: `"Maestrías virtuales"` |
| `organismo` | Sí | `"Servicio Alemán de Intercambio Académico (DAAD)"` |
| `destino` | Sí | `"Alemania"` |
| `tipo` | Sí | `"Maestría"`, `"Movilidad docente"`, `"Estancia de investigación"` |
| `para` | Sí | Lista con uno o más de: `grado`, `egresados`, `posgrado`, `docentes`, `investigadores`, `funcionarios` |
| `areas` | Sí | Lista con `todas`, o uno o más de: `salud`, `social`, `tecnologias` |
| `resumen` | Sí | Una o dos frases |
| `nota` | No | Advertencias o detalles prácticos |
| `cierre` | Uno de los dos | Fecha de cierre `"2027-03-11"`. Con 14 días o menos aparece como «Cierra pronto» |
| `cierreTexto` | Uno de los dos | Cuando no hay fecha única: `"Según el programa, entre diciembre y enero"` |
| `url` | Sí | Enlace a las bases oficiales |

**Cuidado con el formato:** cada texto va entre comillas dobles y las entradas se separan con comas, pero la última de la lista no lleva coma al final. Si el sitio muestra «No pudimos cargar las convocatorias», casi siempre falta o sobra una coma. Para encontrar el error:

```
python -m json.tool convocatorias.json
```

## Probar el sitio en tu computadora

Como las convocatorias se cargan desde un archivo aparte, abrir `index.html` con doble clic no funciona. En la carpeta del sitio, ejecutá:

```
python -m http.server
```

y abrí http://localhost:8000 en el navegador.
