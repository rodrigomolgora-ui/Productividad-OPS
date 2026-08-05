# Configuración — Reporte de Productividad OPS

## Repositorio GitHub
- **URL:** https://github.com/rodrigomolgora-ui/Productividad-OPS
- **GitHub Pages:** https://rodrigomolgora-ui.github.io/Productividad-OPS
- **Archivo a actualizar:** `index.html`

## También disponible en GRID (MeLi interno)
- **Link fijo:** https://grid.adminml.com/d/01KVRDGFC3JXYVXNE25VY10514/view
- **Subir nueva versión vía API** (desde browser autenticado en grid.adminml.com):
```javascript
const blob = new Blob([htmlContent], { type: 'text/html' });
const formData = new FormData();
formData.append('file', blob, 'index.html');
await fetch('/api/v1/documents/01KVRDGFC3JXYVXNE25VY10514/versions', {
  method: 'POST', body: formData
});
```

## Turno
- **Horario:** 05:40 a 15:00
- **Descarga:** al finalizar el turno (~15:00 hrs)

## Secuencia de descarga de CSVs

| # | Proceso | URL | Subpestaña |
|---|---------|-----|------------|
| 0 | Pack    | https://lms.adminml.com/process-monitor?process=packing | — |
| 1 | P2S     | https://lms.adminml.com/process-monitor?process=picking | Pick To Ship |
| 2 | Picking | https://lms.adminml.com/process-monitor?process=picking | Picking |
| 3 | Batch   | https://lms.adminml.com/process-monitor?process=putwallin | Batch sorter |
| 4 | Wallin  | https://lms.adminml.com/process-monitor?process=putwallin | Putwallin |
| 5 | HU      | https://lms.adminml.com/process-monitor?process=hu_assembly | — |

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

> **Si ya hay descargas previas del mismo día**, los números serán más altos (6), (7), etc.
> Siempre tomar los **6 más recientes en orden cronológico** usando Fury CLI:
> ```powershell
> Get-ChildItem "C:\Users\rmolgora\Downloads\Lista de representantes*" | Sort-Object LastWriteTime -Descending | Select-Object -First 6
> ```

> **Si hay menos de 6 archivos del día**, verificar cuál proceso falta comparando tamaños
> en bytes — archivos con el mismo tamaño son duplicados (mismo proceso descargado dos veces).

## Columnas relevantes del CSV

| Campo CSV | Descripción |
|-----------|-------------|
| `Representantes` | Nombre del representante |
| `Prod. neta sist.` | Productividad neta |
| `Utilización` | % de utilización |
| `Tiempo en proceso sist.` | Tiempo en proceso |
| `Unidades` | Unidades procesadas |

## ⚠️ Regla crítica para procesar los datos

Un mismo representante puede aparecer en **múltiples archivos CSV** a lo largo del día.
La regla es:

1. **Mismo nombre + mismo proceso** en varios archivos → tomar el registro con **mayor `Prod. neta sist.`** (es el más actualizado del turno). Descartar los demás.

2. **Mismo nombre + distinto proceso** → incluir **una fila por cada proceso** en que aparezca. Un representante puede haber trabajado en Pack Y en Picking, y debe aparecer en ambos.

**Ejemplo:**
- Marcos en archivo (2) con Picking prod=117 → fila Picking
- Marcos en archivo (3) con Batch prod=176 → fila Batch separada
- Marcos en archivo (4) con Batch prod=535 → reemplaza la fila Batch anterior (mayor prod)
- Resultado: Marcos aparece 2 veces: Picking(117) y Batch(535) ✅

## Herramientas disponibles

- **Fury CLI remote Windows** → leer archivos CSV desde `C:\Users\rmolgora\Downloads\`
- **Claude in Chrome** → navegar LMS y descargar CSVs automáticamente si el usuario lo pide

## Comando para leer todos los CSVs de una vez (Fury CLI / PowerShell)

```powershell
$base = "C:\Users\rmolgora\Downloads\Lista de representantes 2026-MM-DD al 2026-MM-DD de 05_40_00 a 15_00_00"
$files = @("$base.csv","$base (1).csv","$base (2).csv","$base (3).csv","$base (4).csv","$base (5).csv")
foreach ($f in $files) {
  Write-Output "=== $f ==="
  Get-Content $f -Encoding UTF8
  Write-Output ""
}
```

## Formato del index.html

El archivo debe generarse en `C:\Users\rmolgora\Downloads\index.html`.

### Características del HTML:
- Fondo oscuro (`#0f1117`), fuentes DM Sans + DM Mono (Google Fonts)
- **4 tarjetas de stats** arriba: Representantes, Prod. neta prom., Utilización prom., Total unidades
- **Filtros por proceso** con colores:
  - Pack → azul `#60a5fa`
  - P2S → violeta `#a78bfa`
  - Picking → verde `#34d399`
  - Batch → naranja `#fb923c`
  - Wallin → rosa `#f472b6`
  - HU → amarillo `#facc15`
- **Búsqueda** por nombre de representante
- **Tabla ordenable** por cualquier columna (click en header)
- **Barra de utilización** con color semáforo: ≥85% verde, ≥70% naranja, <70% rojo
- **Footer** con promedios del filtro activo
- Columnas: Representante | Proceso | Prod. neta | Utilización | T. proceso | Unidades

### Template HTML completo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Reporte Productividad OPS</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
:root{--bg:#0f1117;--bg2:#181c27;--bg3:#1e2333;--border:rgba(255,255,255,0.07);--border2:rgba(255,255,255,0.12);--text:#e8eaf0;--text2:#8b909e;--text3:#555b6b;}
*{box-sizing:border-box;margin:0;padding:0;}
body{font-family:'DM Sans',sans-serif;background:var(--bg);color:var(--text);min-height:100vh;padding:2rem 1.5rem;}
.container{max-width:1000px;margin:0 auto;}
.header-top{display:flex;align-items:flex-start;justify-content:space-between;flex-wrap:wrap;gap:1rem;margin-bottom:1.5rem;}
.title{font-size:22px;font-weight:600;letter-spacing:-0.5px;}
.subtitle{font-size:12px;color:var(--text2);margin-top:3px;}
.date-badge{font-size:12px;font-weight:500;padding:6px 14px;border-radius:20px;border:1px solid var(--border2);color:var(--text2);background:var(--bg3);}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:10px;margin-bottom:1.5rem;}
.stat{background:var(--bg2);border:1px solid var(--border);border-radius:12px;padding:14px 16px;position:relative;overflow:hidden;}
.stat::before{content:'';position:absolute;top:0;left:0;right:0;height:2px;background:linear-gradient(90deg,#3b82f6,transparent);opacity:0.5;}
.stat-label{font-size:11px;color:var(--text2);text-transform:uppercase;letter-spacing:.6px;margin-bottom:6px;}
.stat-val{font-size:24px;font-weight:600;letter-spacing:-0.5px;}
.controls{display:flex;gap:10px;flex-wrap:wrap;align-items:center;margin-bottom:1rem;}
.filters{display:flex;gap:6px;flex-wrap:wrap;}
.fb{font-family:'DM Sans',sans-serif;font-size:11px;font-weight:500;padding:5px 14px;border-radius:20px;border:1px solid var(--border2);background:transparent;color:var(--text2);cursor:pointer;transition:all .15s;}
.fb:hover{color:var(--text);border-color:rgba(255,255,255,0.2);}
.fb.active{background:rgba(255,255,255,0.08);color:var(--text);border-color:rgba(255,255,255,0.2);}
.search-wrap{flex:1;min-width:200px;}
.search-wrap input{font-family:'DM Sans',sans-serif;width:100%;font-size:13px;padding:7px 12px;border-radius:8px;border:1px solid var(--border2);background:var(--bg2);color:var(--text);outline:none;}
.search-wrap input:focus{border-color:#3b82f6;}
.search-wrap input::placeholder{color:var(--text3);}
.table-wrap{overflow-x:auto;border-radius:12px;border:1px solid var(--border);}
table{width:100%;border-collapse:collapse;font-size:13px;}
thead tr{border-bottom:1px solid var(--border2);}
th{padding:10px 14px;font-size:11px;font-weight:500;color:var(--text3);text-align:left;white-space:nowrap;cursor:pointer;user-select:none;text-transform:uppercase;letter-spacing:.5px;background:var(--bg2);}
th.sorted{color:#3b82f6;}th.right{text-align:right;}
.sa{font-size:10px;margin-left:3px;opacity:.5;}th.sorted .sa{opacity:1;}
tbody tr{border-bottom:1px solid var(--border);}
tbody tr:last-child{border-bottom:none;}tbody tr:hover{background:rgba(255,255,255,0.025);}
td{padding:11px 14px;vertical-align:middle;background:var(--bg2);}td.right{text-align:right;}
.rep-name{font-weight:500;}
.proc-badge{display:inline-block;font-size:10px;font-weight:600;padding:3px 10px;border-radius:20px;text-transform:uppercase;}
.util-row{display:flex;align-items:center;gap:8px;}
.util-bar{flex:1;height:4px;border-radius:2px;background:rgba(255,255,255,0.07);overflow:hidden;min-width:50px;}
.util-fill{height:100%;border-radius:2px;}
.util-v{font-family:'DM Mono',monospace;font-size:11px;font-weight:500;min-width:40px;text-align:right;}
.mono{font-family:'DM Mono',monospace;font-size:11px;color:var(--text2);}
tfoot td{background:var(--bg3);font-size:11px;font-weight:500;color:var(--text2);border-top:1px solid var(--border2);padding:9px 14px;}
.empty{text-align:center;padding:3rem;color:var(--text3);font-size:13px;}
</style>
</head>
<body>
<div class="container">
<div class="header-top">
  <div><div class="title">Reporte de productividad</div><div class="subtitle" id="rc"></div></div>
  <div class="date-badge" id="db"></div>
</div>
<div class="stats">
  <div class="stat"><div class="stat-label">Representantes</div><div class="stat-val" id="s1"></div></div>
  <div class="stat"><div class="stat-label">Prod. neta prom.</div><div class="stat-val" id="s2"></div></div>
  <div class="stat"><div class="stat-label">Utilización prom.</div><div class="stat-val" id="s3"></div></div>
  <div class="stat"><div class="stat-label">Total unidades</div><div class="stat-val" id="s4"></div></div>
</div>
<div class="controls">
  <div class="filters" id="flt"></div>
  <div class="search-wrap"><input type="search" id="srch" placeholder="Buscar representante..." oninput="apl()"></div>
</div>
<div class="table-wrap"><table>
  <thead><tr>
    <th style="width:210px" onclick="srt('nombre')">Representante <span class="sa" id="a0">↕</span></th>
    <th style="width:90px" onclick="srt('proceso')">Proceso <span class="sa" id="a1">↕</span></th>
    <th class="right" style="width:85px" onclick="srt('prod')">Prod. neta <span class="sa" id="a2">↕</span></th>
    <th style="width:130px" onclick="srt('util')">Utilización <span class="sa" id="a3">↕</span></th>
    <th style="width:95px" onclick="srt('tiempo')">T. proceso <span class="sa" id="a4">↕</span></th>
    <th class="right" style="width:85px" onclick="srt('unidades')">Unidades <span class="sa" id="a5">↕</span></th>
  </tr></thead>
  <tbody id="tb"></tbody>
  <tfoot><tr><td colspan="2">Promedio / Total visible</td><td style="text-align:right" id="fp"></td><td id="fu"></td><td></td><td style="text-align:right" id="fuu"></td></tr></tfoot>
</table></div>
</div>
<script>
const FECHA="DD mes YYYY — Turno 1A  05:40 a 15:00";
const D=[
// ["Nombre","Proceso",prod,util,"HH:MM:SS",unidades],
// Proceso: "Pack" | "P2S" | "Picking" | "Batch" | "Wallin" | "HU"
];
const PROC_COLORS={Pack:['#60a5fa','rgba(96,165,250,0.15)','rgba(96,165,250,0.3)'],P2S:['#a78bfa','rgba(167,139,250,0.15)','rgba(167,139,250,0.3)'],Picking:['#34d399','rgba(52,211,153,0.15)','rgba(52,211,153,0.3)'],Batch:['#fb923c','rgba(251,146,60,0.15)','rgba(251,146,60,0.3)'],Wallin:['#f472b6','rgba(244,114,182,0.15)','rgba(244,114,182,0.3)'],HU:['#facc15','rgba(250,204,21,0.15)','rgba(250,204,21,0.3)']};
function gc(p){return PROC_COLORS[p]||['#8b909e','rgba(139,144,158,0.15)','rgba(139,144,158,0.3)'];}
function utilColor(u){return u>=85?'#34d399':u>=70?'#fb923c':'#f87171';}
let data=D.map(r=>({nombre:r[0],proceso:r[1],prod:r[2],util:r[3],tiempo:r[4],unidades:r[5]}));
let af='all',sv='',sk='prod',sa=false;
const KS=['nombre','proceso','prod','util','tiempo','unidades'];
const AS=['a0','a1','a2','a3','a4','a5'];
function init(){
  document.getElementById('rc').textContent=FECHA;
  document.getElementById('db').textContent=FECHA.split('—')[0].trim();
  const prods=data.map(r=>r.prod).filter(v=>v>0);
  const utils=data.map(r=>r.util).filter(v=>v>0);
  document.getElementById('s1').textContent=data.length;
  document.getElementById('s2').textContent=prods.length?(prods.reduce((a,b)=>a+b,0)/prods.length).toFixed(0):'-';
  document.getElementById('s3').textContent=utils.length?(utils.reduce((a,b)=>a+b,0)/utils.length).toFixed(1)+'%':'-';
  document.getElementById('s4').textContent=data.reduce((a,r)=>a+r.unidades,0).toLocaleString('es-MX');
  const procs=['all',...Object.keys(PROC_COLORS)];
  const flt=document.getElementById('flt');
  procs.forEach(p=>{const b=document.createElement('button');b.className='fb'+(p==='all'?' active':'');b.dataset.f=p;b.textContent=p==='all'?'Todos':p;if(p!=='all'){const[c,bg,br]=gc(p);b.style.color=c;b.style.borderColor=br;}b.onclick=()=>fil(b);flt.appendChild(b);});
  data.sort((a,b)=>b.prod-a.prod);ren();
}
function vis(){return data.filter(r=>(af==='all'||r.proceso===af)&&(!sv||r.nombre.toLowerCase().includes(sv)));}
function ren(){
  const rows=vis();const tb=document.getElementById('tb');
  if(!rows.length){tb.innerHTML='<tr><td colspan="6" class="empty">Sin resultados</td></tr>';return;}
  tb.innerHTML=rows.map(r=>{const[c,bg,br]=gc(r.proceso);const uc=utilColor(r.util);const up=Math.min(r.util,100);return '<tr><td><span class="rep-name">'+r.nombre+'</span></td><td><span class="proc-badge" style="color:'+c+';background:'+bg+';border:1px solid '+br+'">'+r.proceso+'</span></td><td class="right"><span class="mono">'+r.prod+'</span></td><td><div class="util-row"><div class="util-bar"><div class="util-fill" style="width:'+up+'%;background:'+uc+'"></div></div><span class="util-v" style="color:'+uc+'">'+(r.util>0?r.util.toFixed(1)+'%':'—')+'</span></div></td><td><span class="mono">'+r.tiempo+'</span></td><td class="right"><span class="mono">'+r.unidades.toLocaleString('es-MX')+'</span></td></tr>';}).join('');
  const vp=rows.map(r=>r.prod).filter(v=>v>0);const vu=rows.map(r=>r.util).filter(v=>v>0);
  document.getElementById('fp').textContent=vp.length?(vp.reduce((a,b)=>a+b,0)/vp.length).toFixed(0):'-';
  document.getElementById('fu').textContent=vu.length?(vu.reduce((a,b)=>a+b,0)/vu.length).toFixed(1)+'%':'-';
  document.getElementById('fuu').textContent=rows.reduce((a,r)=>a+r.unidades,0).toLocaleString('es-MX');
}
function fil(btn){document.querySelectorAll('.fb').forEach(b=>{b.classList.remove('active');b.style.background='';b.style.color='';b.style.borderColor='';});af=btn.dataset.f;if(af==='all')btn.classList.add('active');else{const[c,bg,br]=gc(af);btn.style.background=bg;btn.style.color=c;btn.style.borderColor=br;}apl();}
function apl(){sv=document.getElementById('srch').value.toLowerCase().trim();ren();}
function srt(key){sa=sk===key?!sa:true;sk=key;AS.forEach(id=>{const e=document.getElementById(id);if(e){e.textContent='↕';e.closest('th').classList.remove('sorted');}});const idx=KS.indexOf(key);if(idx>=0){const e=document.getElementById(AS[idx]);if(e){e.textContent=sa?'↑':'↓';e.closest('th').classList.add('sorted');}}data.sort((a,b)=>{let va=a[key],vb=b[key];if(typeof va==='string'){va=va.toLowerCase();vb=vb.toLowerCase();}return sa?(va>vb?1:va<vb?-1:0):(va<vb?1:va>vb?-1:0);});ren();}
init();
</script>
</body>
</html>
```

## Flujo completo cada día

```
1. Usuario escribe "Genera el reporte" o "Ya tienes los archivos"
2. Claude lee con Fury CLI los 6 CSVs más recientes de ~/Downloads
3. Aplica regla nombre+proceso para deduplicar correctamente
4. Genera index.html en C:\Users\rmolgora\Downloads\index.html
5. Usuario sube el index.html a GitHub Pages o GRID
```

## Notas operativas
- Turno 1A tiene más representantes que 1B
- HU puede no tener registros en turno 1B
- Si un proceso no tiene actividad, omitirlo del reporte
- Si hay archivos duplicados (mismo tamaño en bytes), identificar cuál proceso falta y pedirlo
- Los representantes pueden aparecer con distintos "Estado de proceso" dentro del mismo proceso (ej: "Soporte de Picking" sigue siendo Picking)
