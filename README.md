# MineBalance 🪨⚙️
### Sistema Inteligente de Optimización de Destino de Mineral por Volquete en Tiempo Real

> **NSR · Monte Carlo · Factor de Corrección Adaptativo**  
> Universidad Nacional del Altiplano — Escuela Profesional de Ingeniería de Minas  
> Noveno Semestre · Área: Operaciones Mineras · 2026  
> **Autor:** Machaca Espinoza, Jkacson Ruso

---

## 📋 Descripción

**MineBalance** es una herramienta de análisis cuantitativo desarrollada en Python para optimizar en tiempo real el destino de cada volquete extraído en minas subterráneas de Au-Ag. El sistema integra tres componentes principales:

- **Cálculo determinístico del NSR** (Net Smelter Return): valor económico neto por volquete según ruta de procesamiento (cianuración, heap leach o botadero).
- **Simulación Monte Carlo**: evaluación probabilística de la incertidumbre en la ley estimada del muestreo de frente (10,000 iteraciones por defecto).
- **Factor de corrección adaptativo (FC_lab)**: mecanismo de aprendizaje EWMA que reduce el error de estimación guardia a guardia, labor por labor.

---

## 🎯 Criterio de Decisión

| Destino | Condición |
|---|---|
| **Cianuración (CIL/CIP)** | NSR_cian > NSR_heap y L_Au ≥ CO_cian |
| **Heap Leach** | NSR_heap > 0 y L_Au ≥ CO_heap |
| **Botadero** | Ninguna ruta es económicamente viable |

---

## 🗂️ Estructura del Repositorio

```
minebalance/
├── app.py               # Script principal — análisis completo
├── requirements.txt     # Dependencias del proyecto
└── README.md            # Documentación del proyecto
```

### Salidas generadas automáticamente en `Descargas/resultados_minebalance/`

```
resultados_minebalance/
├── MineBalance_Resultados.xlsx     # Excel con 5 hojas de resultados
├── fig1_nsr_histogram.png          # Histograma NSR Monte Carlo
├── fig2_nsr_cdf.png                # Función de distribución acumulada
├── fig3_tornado.png                # Diagrama de tornado — sensibilidad
├── fig4_scatter_au.png             # Scatter Ley Au vs NSR
├── fig5_scatter_ag.png             # Scatter Ley Ag vs NSR
├── fig6_grade_distribution.png     # Distribución de leyes simuladas
├── fig7_nsr_comparison.png         # Comparación NSR determinístico
├── fig8_guard_balance.png          # Balance de guardia completa
├── fig9_fc_ewma.png                # Evolución del factor EWMA
├── fig10_polar_sensitivity.png     # Diagrama polar de sensibilidad
└── fig11_nsr_boxplot.png           # Boxplot comparativo NSR
```

---

## 🧮 Formulaciones Implementadas

### 1. NSR por Volquete
```
NSR_ruta = T × [(L_Au × Rec_Au × P_Au) + (L_Ag × Rec_Ag × P_Ag)] × (1 - D_f)
           - T × (C_p + C_t)
```

### 2. Simulación Monte Carlo
```
L_Au,i ~ N(L̂_Au × FC_lab, σ_m)     para i = 1, 2, ..., N
L_Au,i = max(0, L_Au,i)              [truncación: ley ≥ 0]

E[NSR] = (1/N) × Σ NSR(L_Au,i, L_Ag,i, T, Rec, P, C)
IC 90% = [ P10(NSR_i), P90(NSR_i) ]
```

### 3. Factor de Corrección Adaptativo EWMA
```
E_j     = L_Au,real,j / L̂_Au,j
w_j     = 0.5^(M - j)          [guardias recientes pesan más]
FC_lab  = Σ(E_j × w_j) / Σ(w_j)
```

### 4. Balance de Guardia
```
MF_Au       = Σ T_k × L_Au,real,k × Rec_Au,ruta_k     [gramos]
MF_Ag       = Σ T_k × L_Ag,real,k × Rec_Ag,ruta_k     [gramos]
Eficiencia  = (NSR_total_real / NSR_total_opt) × 100   [%]
```

---

## ⚙️ Instalación y Uso

### 1. Clonar el repositorio
```bash
git clone https://github.com/tu_usuario/minebalance.git
cd minebalance
```

### 2. Instalar dependencias
```bash
pip install -r requirements.txt
```

### 3. Ejecutar
```bash
python app.py
```

El script abre una **interfaz interactiva por consola** que guía al usuario paso a paso. En cada parámetro se muestran valores típicos de referencia. Presiona **ENTER** para usar el valor por defecto.

---

## 📦 Dependencias

| Librería | Versión mínima | Uso en el sistema |
|---|---|---|
| `numpy` | 1.24.0 | Cálculos vectoriales y simulación Monte Carlo |
| `matplotlib` | 3.7.0 | Generación de los 11 gráficos de análisis |
| `openpyxl` | 3.1.0 | Exportación del Excel con 5 hojas estructuradas |
| `scipy` | 1.11.0 | Distribuciones estadísticas y funciones de probabilidad |

> Las librerías estándar de Python (`os`, `sys`, `pathlib`, `sqlite3`, `warnings`, `json`, `datetime`) no requieren instalación. El script **no utiliza pandas**.

---

## 📊 Parámetros de Entrada

### Sección A — Datos del Volquete y Muestreo
| Parámetro | Símbolo | Unidad | Rango típico |
|---|---|---|---|
| Tonelaje por viaje | T | t | 5 – 30 t |
| Ley Au estimada (muestreo) | L̂_Au | g/t | 0.5 – 12.0 g/t |
| Ley Ag estimada (muestreo) | L̂_Ag | g/t | 5 – 200 g/t |
| Error estándar de muestreo | σ_m | g/t | 10 – 50% de L̂_Au |
| Factor de corrección labor | FC_lab | adim. | 0.70 – 1.20 |

### Sección B — Precios de Metales
| Parámetro | Símbolo | Unidad | Rango típico |
|---|---|---|---|
| Precio del oro | P_Au | $/g | 33 – 62 $/g |
| Precio de la plata | P_Ag | $/g | 0.30 – 0.65 $/g |
| Deducción por fundición | D_f | % | 3 – 10% |

### Sección C — Rutas de Procesamiento
| Parámetro | Cianuración | Heap Leach |
|---|---|---|
| Recuperación Au | 88 – 96% | 45 – 75% |
| Recuperación Ag | 70 – 90% | 20 – 55% |
| Costo de procesamiento | 120 – 250 $/t | 35 – 100 $/t |
| Costo de transporte | 5 – 30 $/t | 3 – 15 $/t |
| Cut-off grade mínimo Au | 1.0 – 4.0 g/t | 0.4 – 2.0 g/t |

---

## 📚 Referencias Bibliográficas

- Hustrulid, W. & Kuchta, M. (2006). *Open Pit Mine Planning and Design*. 2nd Ed. Taylor & Francis, London.
- SME Mining Engineering Handbook (2011). 3rd Ed. Society for Mining, Metallurgy and Exploration.
- Pitard, F.F. (1993). *Pierre Gy's Sampling Theory and Sampling Practice*. 2nd Ed. CRC Press.
- Hammersley, J.M. & Handscomb, D.C. (1964). *Monte Carlo Methods*. Methuen & Co., London.
- Ang, A.H. & Tang, W.H. (1984). *Probability Concepts in Engineering Planning and Design*. Vol. 1. Wiley.
- Brown, R.G. (1959). *Statistical Forecasting for Inventory Control*. McGraw-Hill. [EWMA]
- Carrasco, P. & Jélvez, E. (2012). Reconciliation in underground mining operations. *Int. J. Mining Sci. Technol.*, 22(5), 691–697.
- LBMA — London Bullion Market Association (2025). Gold and Silver Price Statistics. https://www.lbma.org.uk

---

## 📄 Licencia

Proyecto académico — Universidad Nacional del Altiplano, Puno, Perú.  
Uso libre para fines educativos e investigación en ingeniería de minas.
