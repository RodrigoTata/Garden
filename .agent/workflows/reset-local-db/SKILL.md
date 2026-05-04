---
name: reset-local-db
description: Limpieza selectiva y reversible de la base de datos LOCAL (localhost). Elimina datos operativos (dispensaciones, cultivos, inventario) y preserva datos maestros (usuarios, médicos, recetas, documentos). Usa cuando quieras preparar el sistema para una prueba desde cero sin perder los registros de socios.
---

# Reset Local DB (Limpieza Selectiva Reversible)

Prepara la base de datos **local** para una sesión de prueba limpia. Siempre exporta un backup antes de borrar. El proceso es reversible en cualquier momento.

> **IMPORTANTE — Ambiente objetivo**: Este skill opera EXCLUSIVAMENTE sobre la base de datos local (`localhost`). El backend local se levanta con `npm run start:local` (usa `.env.local`). Si el backend activo fue iniciado con `npm run start:dev`, está apuntando a **GCP** y este skill NO lo tocará.

> **No interrumpas el proceso a mitad.** Backup → Verificar → Limpiar → Confirmar. Si algo falla en cualquier paso, detente y haz ROLLBACK.

---

## Tablas Protegidas (nunca se tocan)

| Tabla | Razón |
|---|---|
| `usuarios` | Datos maestros de socios |
| `documentos_usuario` | Carnet, recetas de ingreso, documentos de identidad |
| `receta_medica` | Historial médico del socio |
| `medico` | Catálogo de médicos registrados |
| `membresia` | Estado de membresía activa del socio |
| `strains` | Catálogo de variedades |
| `products` | Catálogo de productos |
| `product_ingredients` | Composición de productos |
| `offers` | Ofertas del catálogo |

## Tablas a Limpiar (datos operativos)

| Tabla | Razón |
|---|---|
| `dispensacion` | Historial de dispensaciones realizadas |
| `dispensacion_order` | Órdenes de dispensación |
| `dispensacion_offers` | Relación dispensación-oferta |
| `donacion` | Donaciones registradas |
| `lotes_cultivo` | Lotes de cultivo activos |
| `unidades_cultivo` | Unidades de cultivo |
| `plantas` | Plantas individuales |
| `eventos_cultivo` | Historial de eventos de cultivo |
| `inventario` | Stock del inventario general |
| `uso_inventario` | Uso del inventario |
| `store_stock` | Stock de la tienda |
| `recarga_estanque` | Historial de recargas |
| `formula` | Fórmulas de fertilización |
| `recetas_fertilizacion` | Recetas de fertilización |
| `componentes_receta` | Componentes de recetas de fertilización |
| `trigger_production` | Triggers de producción |
| `ticket` | Tickets de soporte |
| `ticket_mensaje` | Mensajes de tickets |
| `recordatorios` | Recordatorios programados |

---

## Proceso

### Paso 1 — Verificar el ambiente correcto

Antes de cualquier acción, confirmar que el backend local está apuntando a `localhost` y no a GCP.

```bash
# Verificar qué .env usa el script de backup (debe ser .env.local)
# El comando de arranque correcto del backend local es:
npm run start:local   # NODE_ENV=local → .env.local → localhost
```

Revisar `.env.local` y confirmar:
- `DB_HOST=localhost`
- `DB_PORT=5432`

Si `DB_HOST` apunta a una IP pública (ej. `34.28.x.x`), **DETENER**. No proceder.

---

### Paso 2 — Backup automático (Red de seguridad)

Crear el script `scratch/backup_local_db.ts` si no existe, y ejecutarlo.

El script debe:
1. Conectarse a la DB local usando credenciales de `.env.local`.
2. Exportar todas las **tablas protegidas** a un archivo JSON en `scratch/backups/`.
3. El nombre del archivo debe incluir la fecha y hora: `backup_local_YYYY-MM-DD_HH-mm.json`.
4. Imprimir cuántas filas exportó de cada tabla.

```typescript
// Estructura del archivo de backup generado:
{
  "timestamp": "2026-05-03T02:00:00.000Z",
  "usuarios": [...],
  "documentos_usuario": [...],
  "receta_medica": [...],
  "medico": [...],
  "membresia": [...]
}
```

**No continuar si el backup falla.**

---

### Paso 3 — Verificación pre-limpieza

Antes de ejecutar el TRUNCATE, mostrar un conteo de todas las tablas involucradas.

```typescript
// El script debe imprimir algo así:
// ✅ PROTEGIDAS (no se tocarán):
//   usuarios:           12 filas
//   documentos_usuario: 8 filas
//   receta_medica:      23 filas
//   medico:             5 filas
//   membresia:          12 filas
//
// 🗑️  A LIMPIAR:
//   dispensacion:       47 filas
//   lotes_cultivo:      3 filas
//   inventario:         15 filas
//   ... (resto de tablas)
```

Pedir confirmación explícita al usuario antes de proceder. Mostrar el mensaje:

> "¿Confirmas la limpieza? Las filas protegidas se mantienen. Esta acción no se puede deshacer sin el backup."

---

### Paso 4 — Limpieza con transacción

Ejecutar el TRUNCATE dentro de una transacción SQL para poder hacer ROLLBACK si algo falla.

```sql
BEGIN;

-- Solo tablas operativas, en orden respetando FK constraints
TRUNCATE TABLE 
  dispensacion_offers,
  dispensacion_order,
  dispensacion,
  donacion,
  uso_inventario,
  store_stock,
  recarga_estanque,
  trigger_production,
  formula,
  componentes_receta,
  recetas_fertilizacion,
  eventos_cultivo,
  plantas,
  unidades_cultivo,
  lotes_cultivo,
  inventario,
  ticket_mensaje,
  ticket,
  recordatorios
CASCADE;

-- Verificación de integridad post-limpieza
SELECT 'usuarios' as tabla, count(*) FROM usuarios
UNION ALL SELECT 'medicos', count(*) FROM medico
UNION ALL SELECT 'receta_medica', count(*) FROM receta_medica
UNION ALL SELECT 'documentos_usuario', count(*) FROM documentos_usuario;

-- Si los conteos son correctos → COMMIT
COMMIT;

-- Si alguna tabla protegida quedó en 0 → ROLLBACK
-- ROLLBACK;
```

La regla de COMMIT/ROLLBACK:
- Si `usuarios > 0` Y `medico >= 0` Y `receta_medica >= 0` → **COMMIT**
- Si `usuarios = 0` (nunca debería ocurrir) → **ROLLBACK** inmediato

---

### Paso 5 — Reset de secuencias (Opcional)

Si quieres que los folios de dispensación vuelvan a empezar desde 1 para la prueba:

```sql
-- Solo ejecutar si el usuario lo solicita explícitamente
ALTER SEQUENCE IF EXISTS dispensacion_folio_seq RESTART WITH 1;
```

> **Guardrail**: No resetear secuencias si hay datos en las tablas de médicos o recetas que referencian dispensaciones antiguas por folio.

---

### Paso 6 — Verificación final

Ejecutar el script de conteo una vez más y mostrar el estado limpio del sistema.

```
✅ LIMPIEZA COMPLETADA
  Tablas protegidas intactas:
    usuarios:           12 filas ✓
    documentos_usuario: 8 filas ✓
    receta_medica:      23 filas ✓
    medico:             5 filas ✓
  
  Tablas limpiadas:
    dispensacion:       0 filas ✓
    inventario:         0 filas ✓
    plantas:            0 filas ✓
    ... 
  
  Backup disponible en: scratch/backups/backup_local_2026-05-03_02-00.json
```

---

## Cómo Revertir

Si después de la limpieza necesitas restaurar los datos:

```bash
# Ejecutar el script de restore apuntando al backup generado
npx cross-env NODE_ENV=local ts-node scratch/restore_local_db.ts scratch/backups/backup_local_YYYY-MM-DD_HH-mm.json
```

El script de restore debe:
1. Leer el archivo JSON indicado.
2. Insertar los registros en las tablas protegidas usando `INSERT ... ON CONFLICT DO NOTHING` para evitar duplicados.
3. Imprimir cuántas filas restauró de cada tabla.

---

## Scripts de Apoyo

| Script | Propósito |
|---|---|
| `scratch/backup_local_db.ts` | Exportar tablas protegidas a JSON |
| `scratch/restore_local_db.ts` | Re-importar desde un JSON de backup |
| `scratch/check_counts.ts` | Ver conteo de todas las tablas |
| `scratch/db_cleanup.ts` | Limpieza (usar solo a través de este skill) |

---

## Guardrails

- **Nunca ejecutar sin backup previo.** El Paso 2 es obligatorio, no opcional.
- **Nunca usar `npm run start:dev` como indicador del ambiente local.** `start:dev` apunta a GCP.
- **Verificar `DB_HOST` en el `.env` activo** antes de cualquier operación destructiva.
- **No borrar `receta_medica` ni `medico`.** Aunque no tengan dispensaciones asociadas, son datos maestros del socio.
- **No asumir nombres de tablas en singular.** Este proyecto usa nombres en plural (`usuarios`, `dispensaciones`). Siempre verificar con `SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'`.
- **No hacer COMMIT automático.** El script debe mostrar el conteo de verificación y esperar confirmación antes de hacer COMMIT.
