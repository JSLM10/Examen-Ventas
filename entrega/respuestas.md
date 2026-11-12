## 1. ¿Por qué usar `_loaded_at` y no `fecha_venta` para este control?

Porque `fecha_venta` dice cuándo se hizo la venta y `_loaded_at` dice cuándo llegó el dato a `raw`, y lo que quiero saber es si la carga está al día. Las ventas del ejercicio son de enero de 2026, así que si midiera con `fecha_venta` la fuente siempre saldría vencida, aunque la carga fuera de hace dos horas. Y al revés: una venta vieja puede cargarse hoy, y eso es una carga fresca.

Se ve en los escenarios: los datos de negocio fueron los mismos seis registros en los tres, y lo único que cambió fue el último `_loaded_at` (hace unas 2 h, 18 h y 30 h). Solo ese campo diferenció los resultados: pasó, advirtió y falló.

## 2. ¿Por qué las pruebas de integridad no garantizan actualidad?

Porque revisan el contenido de las filas (sin duplicados, sin nulos, valores válidos), pero no el tiempo. Unos datos viejos pero bien formados las pasan igual. En el escenario `actualizada` el `dbt build` pasó todas las pruebas, y con los mismos seis registros cargados hace 30 horas habrían pasado igual, porque el contenido no cambia.

Además `dbt build` no corre `dbt source freshness` por sí solo, así que podría terminar bien reconstruyendo el resumen con información desactualizada. Por eso freshness va como tarea aparte antes de `construir_resumen`: si falla, `construir_resumen` queda en `upstream_failed` y el resumen anterior se conserva.