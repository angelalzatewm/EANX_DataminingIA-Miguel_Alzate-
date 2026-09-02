---

## 1. 🎯 Objetivo y Pregunta Central

### Objetivo General
Diseñar, implementar y desplegar una arquitectura de datos e ingeniería analítica integral de baja latencia[cite: 1, 2]. El sistema orquesta la telemetría financiera en tiempo real de fondos de inversión (**Proyecto TOKEN RIO**)[cite: 1], la automatización conversacional y logística de un servicio de estética canina (**HeisGroomer PetTech**)[cite: 1, 2], y la simulación e ingesta de variables biológicas e hidrológicas para el sector agroindustrial en la región de Urabá (**Urabá AgroTech**)[cite: 1].

### Pregunta Central de Investigación y Desarrollo
> **¿Es posible estructurar un canal unificado de datos e inferencia en tiempo real, capaz de ingerir y modelar series de tiempo financieras altamente estocásticas, logs estandarizados de interacción conversacional (NLP/Regex) y métricas de telemetría IoT heterogéneas, garantizando consistencia relacional (ACID), detección anticipada de anomalías y alta disponibilidad sin incrementar drásticamente la latencia de respuesta?**

---

## 2. 🗄️ Fuente de los Datos y Proceso de Limpieza

### Fuentes de Datos
1. **Historial gastos mensuales** Extracción .csv del departamento de compras


---

### Proceso de Limpieza y ETL (Extract, Transform, Load)

CAPAS DE TRATAMIENTO DE DATOS
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CAPA BRONZE (RAW DATASET)                             │
│   • Registros duplicados (filas y claves primarias 'id_transaccion')        │
│   • Inconsistencias de formato: '$', espacios, 'leet', fechas fuera de ISO  │
│   • Valores faltantes (NaN) e Incoherencias numéricas (signos invertidos)   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PIPELINE DE TRANSFORMACIÓN SILVER                        │
│ 1. Deduplicación por Clave & Contenido                                      │
│ 2. Sanitización RegEx Vectorizada & Standard Parsers                        │
│ 3. Imputación Estadística Inteligente (Mediana por Grupos Operativos)       │
│ 4. Detección y Tratamiento Estadístico de Outliers (Z-Score Robust/MAD, IQR)│
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                 CAPA GOLD & MÉTIRICAS VISUALES DE CALIDAD
  MINERÍA DE DATOS Y  MODELADO DE DATOS
└─────────────────────────────────────────────────────────────────────────────┘

---

## 3. ⛏️ Técnicas de Minería de Datos y ML Aplicadas

### A. Segmentación y Clustering de Transacciones (K-Means)
Aplicación del algoritmo **K-Means** sobre el historial transaccional de compras y balance para identificar patrones de consumo y agrupar clientes según su recurrencia y volumen de capital.

* **Fórmula de Distancia Euclidiana:**
  $$d(x, c_i) = \sqrt{\sum_{j=1}^{n} (x_j - c_{ij})^2}$$

---

### B. Detección de Anomalías y Circuit Breakers (Isolation Forest)
Implementación de **Isolation Forest** para aislar desviaciones operativas en la liquidez y solicitudes atípicas en la API. El algoritmo aísla observaciones creando particiones aleatorias sobre el espacio de características.

* **Función de Puntuación de Anomalía:**
  $$s(x, n) = 2^{-\frac{E(h(x))}{c(n)}}$$

  *(Donde $h(x)$ es la longitud del camino de la muestra $x$, $E(h(x))$ es el promedio en el conjunto de árboles de decisión, y $c(n)$ es la longitud media de búsquedas no exitosas en un árbol binario).*

                 ISOLATION FOREST ANOMALY SCORE
Anomalía Severa (Rechazo)       Operación Normal       Circuit Breaker CapEx
[ 0.0 ────────────────────── 0.5 ────────────────── 0.8 ──────────────────── 1.0 ]

---

## 4. 📈 Resultados Obtenidos y Conclusiones
Gráfico 1: Clusterización K-Means & Anomalías (Isolation Forest)
Diagnóstico Matemático y Visual:
Comportamiento Regular: Los datos se estructuran en dos bandas densas y bien
delimitadas a lo largo de 2026:
Perfil 1 (rojo: de $0 USD a $600 USD) y
Perfil 2 (azul: de $600 USD a $1,200 USD).
Anomalías Extremas (Superior): Transacciones aisladas etiquetadas con marcadores
x cerca del límite de $1,800 USD en Perfil 3 (verde).
Anomalías en Límite Inferior (Atípico Crítico): Se observan puntos etiquetados
con x pegados al eje de $0 USD en el Perfil 1.
Reales Decisiones de Negocio y Arquitectura:
Control Fijo vs. Dinámico (Filtro por Circuit Breakers): Las transacciones atípicas
superiores a $1,800 USD requieren una regla estricta de validación previa mediante
un servicio API.
Auditoría de Micro-transacciones ($0 USD): Las anomalías detectadas cercanas a $0 USD
indican procesamientos incompletos o registros basura en la base de datos que deben
 filtrarse desde la capa de ingesta en la API (FastAPI) o en la base de datos relacional (PostgreSQL).

Gráfico 2: Distribución de Presupuesto por Perfil ML (Boxplot)Diagnóstico Matemático y Visual:

 Perfil 1 (Caja Base): Mediana en $330 USD, con un Rango Intercuartílico ($\text{IQR} = Q_3 - Q_1$)
 de $320 USD ($160 USD a $480 USD). Muestra alta frecuencia transaccional y dispersión controlada.

 Perfil 2 (Caja Intermedia): Mediana en $910 USD, Rango Intercuartílico ($\text{IQR}$)
 de $270 USD ($780 USD a $1,050 USD). Presenta un valor atípico alto aislado cerca de $1,800 USD.

 Perfil 3 (Línea Continua Superior): Representa una categoría colapsada en un único punto/valor
 fijo en torno a los $1,800 USD.
 Reales Decisiones de Negocio Aclaradas:
 Aclaración sobre la Eficiencia Financiera:
 Perfil 1 (Operación Frecuente / Bajo Monto): Al concentrar la mayor cantidad de eventos,
 la meta de optimización no es reducir presupuestos, sino reducir costos transaccionales fijos
 (comisiones bancarias, procesamiento de facturas, pasarelas de pago y consolidación de pedidos semanales/mensuales en un solo pago a proveedores).
 Perfil 2 (Núcleo Operativo): Dado que presenta la mayor carga monetaria agregada y estabilidad en su IQR ($780 USD - $1,050 USD),
 se debe establecer un presupuesto de tesorería mensual protegido que cubra esta variabilidad
 esperada sin requerir aprobaciones manuales adicionales.
 Perfil 3 (Gasto CapEx / Infraestructura Especial): Al no comportarse como un flujo continuo sino como eventos puntuales aislados,
 esta categoría debe extraerse del presupuesto operativo recurrente (OpEx) e integrarse en una bolsa de inversión de capital (CapEx) con aprobación previa obligatoria.

LINK FINAL DEL VIDEO EXPLICATIVO

https://drive.google.com/file/d/1cJaCW2Ts3uW-RvN74g05SXRMXb23pr6r/view?usp=sharing
---
