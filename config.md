# Configuración — Reporte de Productividad OPS

## Repositorio GitHub
- **URL:** https://github.com/rodrigomolgora-ui/Productividad-OPS
- **GitHub Pages:** https://rodrigomolgora-ui.github.io/Productividad-OPS
- **Archivo a actualizar:** `index.html`

## Turno
- **Horario:** 05:40 a 15:00
- **Descarga:** al finalizar el turno (~15:00 hrs)

## Secuencia de descarga de CSVs

El orden en que se descargan los archivos define el proceso. Yo navego y descargo en este orden exacto:

| # | Proceso | URL | Subpestaña |
|---|---------|-----|------------|
| 1 | Pack    | https://lms.adminml.com/process-monitor?process=packing | — |
| 2 | P2S     | https://lms.adminml.com/process-monitor?process=picking | Pick To Ship |
| 3 | Picking | https://lms.adminml.com/process-monitor?process=picking | Picking |
| 4 | Batch   | https://lms.adminml.com/process-monitor?process=putwallin | Batch sorter |
| 5 | Wallin  | https://lms.adminml.com/process-monitor?process=putwallin | Putwallin |
| 6 | HU      | https://lms.adminml.com/process-monitor?process=hu_assembly | — |

## Cómo identificar los archivos descargados

Los archivos caen en `C:\Users\rmolgora\Downloads\` con el nombre:
```
Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00.csv      → Pack
Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00 (1).csv  → P2S
Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00 (2).csv  → Picking
Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00 (3).csv  → Batch
Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00 (4).csv  → Wallin
Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00 (5).csv  → HU
```

> **Importante:** Si ya hay descargas previas del mismo día, los números serán más altos (6), (7), etc. Siempre tomar los **6 más recientes** en orden cronológico.

## Columnas relevantes del CSV

| Campo CSV | Descripción |
|-----------|-------------|
| `Representantes` | Nombre del representante |
| `Prod. neta sist.` | Productividad neta |
| `Utilización` | % de utilización |
| `Tiempo en proceso sist.` | Tiempo en proceso |
| `Unidades` | Unidades procesadas |

## Flujo completo cada día

```
1. Usuario escribe "Genera el reporte"
2. Claude in Chrome navega y descarga los 6 CSVs en orden
3. Fury CLI lee los 6 archivos más recientes de ~/Downloads
4. Claude procesa los datos y genera el index.html
5. Usuario sube el index.html a GitHub
6. El equipo ve el reporte en: rodrigomolgora-ui.github.io/Productividad-OPS
```

## Notas
- El turno 1A tiene más representantes que el turno 1B
- HU puede no tener registros en turno 1B (verificar)
- Si un proceso no tiene actividad, omitirlo del reporte
