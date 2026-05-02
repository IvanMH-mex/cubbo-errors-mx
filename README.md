# Cubbo · Errores y Productividad MX

Reporte semanal **autogenerado** de errores de plataforma y productividad operativa de los almacenes Cubbo en México (Toreo, Plata, Tlalnepantla).

## 🌐 Ver en vivo

→ **[Abrir reporte](https://IvanMH-mex.github.io/cubbo-errors-mx/)**

## ⚙️ Cómo funciona

El reporte se regenera automáticamente cada **lunes a las 8:00 AM CDMX** mediante GitHub Actions:

1. Workflow corre en GitHub (sin intervención manual)
2. `refresh.py` se autentica en Metabase via API REST
3. Ejecuta las queries semanales contra la base productiva
4. Calcula KPIs y reemplaza placeholders en `template.html`
5. Genera `index.html` con datos frescos
6. Auto-commit + auto-deploy en GitHub Pages

**Tu trabajo recurrente:** cero. Solo abres el link cuando quieras revisar.

## 📂 Estructura

```
.
├── index.html                       # Reporte publicado (autogenerado, NO editar a mano)
├── template.html                    # Plantilla con placeholders {{ ... }}
├── refresh.py                       # Script que llama Metabase y regenera index.html
├── requirements.txt                 # Dependencias Python
├── data.json                        # Backup JSON de últimas métricas (audit)
├── 404.html                         # Página de error
├── .github/workflows/refresh.yml    # Cron lunes 8am CDMX
└── README.md
```

## 🔐 Setup inicial (una sola vez)

### 1. Crear el repo en GitHub
- Crea repo público llamado `cubbo-errors-mx`
- Sube todos los archivos de esta carpeta

### 2. Configurar secrets de Metabase
En el repo: **Settings → Secrets and variables → Actions → New repository secret**

Crea estos 4 secrets:

| Nombre | Valor |
|---|---|
| `METABASE_URL` | URL base, ej: `https://metabase.cubbo.com` |
| `METABASE_USER` | Email del usuario Metabase |
| `METABASE_PASS` | Password del usuario Metabase |
| `METABASE_DB_ID` | `2` (o el ID de la base "Cubbo API") |

> 💡 **Recomendación:** Crear un usuario Metabase dedicado tipo `reports@cubbo.com` con permisos de solo-lectura, en vez de usar tus credenciales personales.

### 3. Activar GitHub Pages
**Settings → Pages**:
- Source: **Deploy from a branch**
- Branch: **main** / **/ (root)**
- Save

### 4. Activar GitHub Actions
**Settings → Actions → General**:
- Allow all actions
- En "Workflow permissions": elegir **Read and write permissions**

### 5. Probar manualmente
**Actions → Refresh weekly report → Run workflow → Run**

Si funciona en ~2 minutos verás el commit automático y el reporte actualizado.

## 🛠️ Correr local (para debugging)

```bash
# Configurar variables (una sola vez)
export METABASE_URL="https://metabase.cubbo.com"
export METABASE_USER="tu@email.com"
export METABASE_PASS="tu-password"
export METABASE_DB_ID=2

# Instalar dependencias
pip install -r requirements.txt

# Correr
python refresh.py
```

## 📊 Queries que ejecuta

| ID | Métrica |
|---|---|
| `totales` | Total errores 7d |
| `top_errores` | Top 20 error_class_name |
| `bug_texto_largo` | Q74 — bug `process_action` (texto largo) |
| `plata_card_pct` | Q75 — concentración Plata Card en bug texto largo |
| `shipping_method_1090` | Hallazgo secundario — config faltante |
| `t1_envios_circuit` | Errores T1 Envíos circuit breaker |
| `ordenes_por_wh` | Volumen órdenes shipped por almacén |

## 🚨 Si algo falla

**El workflow no corre el lunes:**
- Ve a Actions → último run → revisa logs
- Causas comunes: secret expirado, password Metabase cambiado, SSL del proveedor

**Reporte sale con números raros:**
- `data.json` tiene el último JSON crudo — ahí puedes ver qué devolvió Metabase
- Corre `refresh.py` localmente para debuggear

**Necesito modificar el contenido (no solo datos):**
- Edita `template.html` — los placeholders `{{ }}` se preservan
- Commit + push → siguiente run automático lo aplica

## 👤 Autor

**Iván Hernández** — Operations · Cubbo Mexico

---

> ⚠️ **No editar `index.html` a mano** — se sobrescribe en cada run del workflow. Toda modificación debe ir en `template.html`.
