# Análisis Técnico de Soluciones de Almacenamiento de Datos — Archivos del proyecto

Este documento contiene todos los archivos listos para subir al repositorio `data-storage-analysis/` en GitHub y desplegar en Streamlit Cloud.

---

## Estructura del proyecto

```
data-storage-analysis/
│
├── app.py                # Aplicación principal en Streamlit
├── requirements.txt      # Librerías necesarias
├── assets/               # Diagramas e imágenes
│   └── infra_diagram.png
└── README.md             # Documentación del proyecto
```

---

## app.py

```python
import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import plotly.graph_objects as go
import plotly.express as px

st.set_page_config(page_title="Análisis Técnico - Soluciones de Almacenamiento", layout="wide")

# --- Descripción del escenario ---
st.title("Análisis Técnico de Soluciones de Almacenamiento de Datos")
st.markdown(
    """
    **Escenario:** Una empresa con crecimiento de datos moderado busca evaluar alternativas de almacenamiento (HDD, SSD, Cintas y Nube). Evaluaremos criterios técnicos — velocidad, capacidad, costo, fiabilidad, consumo, seguridad y escalabilidad — y propondremos una solución.
    """
)

# --- Tabla comparativa (datos ficticios pero realistas) ---
options = [
    {
        'Tecnología': 'HDD',
        'Velocidad_MBps': 200,
        'Capacidad_TB': 12,
        'Costo_USD_per_GB': 0.02,
        'Fiabilidad_score': 6,
        'Consumo_W': 8,
        'Seguridad_score': 6,
        'Escalabilidad_score': 6
    },
    {
        'Tecnología': 'SSD',
        'Velocidad_MBps': 550,
        'Capacidad_TB': 4,
        'Costo_USD_per_GB': 0.10,
        'Fiabilidad_score': 8,
        'Consumo_W': 3,
        'Seguridad_score': 7,
        'Escalabilidad_score': 7
    },
    {
        'Tecnología': 'Cintas',
        'Velocidad_MBps': 150,
        'Capacidad_TB': 30,
        'Costo_USD_per_GB': 0.005,
        'Fiabilidad_score': 7,
        'Consumo_W': 1,
        'Seguridad_score': 5,
        'Escalabilidad_score': 5
    },
    {
        'Tecnología': 'Nube',
        'Velocidad_MBps': 1000,
        'Capacidad_TB': 1000,
        'Costo_USD_per_GB': 0.023,
        'Fiabilidad_score': 9,
        'Consumo_W': 0,
        'Seguridad_score': 8,
        'Escalabilidad_score': 10
    }
]

df = pd.DataFrame(options)

st.subheader("Comparativa técnica: HDD vs SSD vs Cintas vs Nube")
st.dataframe(df.style.format({
    'Velocidad_MBps':'{:.0f}',
    'Capacidad_TB':'{:.0f}',
    'Costo_USD_per_GB':'${:.3f}',
    'Fiabilidad_score':'{:.0f}',
    'Consumo_W':'{:.0f}',
    'Seguridad_score':'{:.0f}',
    'Escalabilidad_score':'{:.0f}'
}))

# --- Visualizaciones ---
st.subheader("Visualizaciones técnicas")
col1, col2 = st.columns(2)

# Bar chart: velocidad y capacidad (matplotlib)
with col1:
    st.write("**Velocidad (MB/s) y Capacidad (TB)**")
    fig, ax = plt.subplots(figsize=(6,3))
    x = np.arange(len(df))
    ax.bar(x - 0.2, df['Velocidad_MBps'], width=0.4, label='Velocidad (MB/s)')
    ax.bar(x + 0.2, df['Capacidad_TB'], width=0.4, label='Capacidad (TB)')
    ax.set_xticks(x)
    ax.set_xticklabels(df['Tecnología'])
    ax.set_ylabel('Valor')
    ax.legend()
    st.pyplot(fig)

# Cost per GB (plotly)
with col2:
    st.write("**Costo por GB (USD)**")
    fig2 = px.bar(df, x='Tecnología', y='Costo_USD_per_GB', text='Costo_USD_per_GB')
    fig2.update_traces(texttemplate='$%{text:.3f}', textposition='outside')
    fig2.update_layout(yaxis_title='USD per GB', uniformtext_minsize=8)
    st.plotly_chart(fig2, use_container_width=True)

# Radar chart for Fiabilidad, Escalabilidad, Seguridad
st.write("**Radar: Fiabilidad - Escalabilidad - Seguridad**")
categories = ['Fiabilidad', 'Escalabilidad', 'Seguridad']

radar_df = pd.DataFrame({
    'Tecnología': df['Tecnología'],
    'Fiabilidad': df['Fiabilidad_score'],
    'Escalabilidad': df['Escalabilidad_score'],
    'Seguridad': df['Seguridad_score']
})

fig3 = go.Figure()
for i, row in radar_df.iterrows():
    fig3.add_trace(go.Scatterpolar(
        r=[row['Fiabilidad'], row['Escalabilidad'], row['Seguridad']],
        theta=categories,
        fill='toself',
        name=row['Tecnología']
    ))

fig3.update_layout(polar=dict(radialaxis=dict(visible=True, range=[0,10])), showlegend=True)
st.plotly_chart(fig3, use_container_width=True)

# --- Simulación de rendimiento / crecimiento de datos ---
st.subheader("Simulación de crecimiento y rendimiento (50% en 2 años)")

# Parámetros de base
initial_data_tb = st.number_input('Datos iniciales (TB):', min_value=1.0, value=50.0, step=1.0)
growth_percent = 50.0  # 50% en 2 años
years = st.slider('Horizonte (años):', 1, 5, 2)

# Modelo simple: crecimiento compuesto anual equivalente para alcanzar 50% en 2 años
annual_rate = (1 + growth_percent/100) ** (1/2) - 1 if years>=2 else (growth_percent/100) / years
annual_rate = float(annual_rate)

# Proyección
timeline = np.arange(0, years+1)
projection = [initial_data_tb * ((1+annual_rate) ** t) for t in timeline]

sim_df = pd.DataFrame({'Año': timeline, 'Datos_TB': projection})

st.write('Tasa anual equivalente usada en simulación: {:.2%}'.format(annual_rate))
st.line_chart(sim_df.rename(columns={'Año':'index'}).set_index('Año'))

# Comparación HDD vs SSD: tiempo de acceso hipotético global para backups/restore
st.write("**Comparación de tiempo estimado de restauración (hipotético)**")

# Supongamos: tiempo proporcional a volumen / velocidad
hdd_speed = df.loc[df['Tecnología']=='HDD','Velocidad_MBps'].values[0]
ssd_speed = df.loc[df['Tecnología']=='SSD','Velocidad_MBps'].values[0]

bytes_per_TB = 1e9  # simplificación: TB -> 1e9 MB for this toy-model (not exact)

hdd_times = [ (d * bytes_per_TB) / hdd_speed / 3600 for d in projection]  # hours
ssd_times = [ (d * bytes_per_TB) / ssd_speed / 3600 for d in projection]

comp_df = pd.DataFrame({'Año': timeline, 'HDD_horas': hdd_times, 'SSD_horas': ssd_times})
st.dataframe(comp_df.style.format({'HDD_horas':'{:.1f}','SSD_horas':'{:.1f}'}))

st.area_chart(comp_df.set_index('Año'))

# --- Diagrama de infraestructura ---
st.subheader("Diagrama de infraestructura")
st.markdown("Imagen estática: `assets/infra_diagram.png`")
try:
    st.image('assets/infra_diagram.png', caption='Infraestructura propuesta (híbrida: SSD + Nube)')
except Exception as e:
    st.info('Coloca `infra_diagram.png` dentro de la carpeta `assets/` en el repo para que se muestre aquí.')

# --- Análisis de riesgos y oportunidades ---
st.subheader("Análisis de riesgos y oportunidades")
st.markdown("**Riesgos técnicos:**")
st.write("- Dependencia de proveedor de nube (lock-in).\n- Costos variables en la nube por e/s y transferencias.\n- Latencia en accesos a datos fríos.\n- Pérdida de datos por fallos hardware si no hay replicación adecuada.")

st.markdown("**Oportunidades:**")
st.write("- Híbrido SSD + nube: rendimiento local para cargas críticas + escalabilidad en nube para archivado.\n- Uso de políticas lifecycle y tiering para optimizar costos.\n- Automatización de backups y replicación multi-zona para alta disponibilidad.")

# --- Conclusiones técnicas ---
st.subheader("Conclusiones técnicas y recomendación")
st.markdown(
    """
    Recomendación: adoptar una **solución híbrida** que combine SSD para cargas de trabajo críticas y la Nube para escalabilidad y archivado.

    Justificación breve:
    - SSD: mayor rendimiento y menor consumo energético para bases de datos/productividad.
    - Nube: escalabilidad y fiabilidad operacional, buenos para archivado y respaldo.
    - Cintas/hardware de archivo: opción muy económica para retención a largo plazo.
    """
)

st.markdown("---")
st.caption('Proyecto preparado para desplegar en Streamlit Cloud — sube este repo a GitHub y conecta en https://streamlit.io/cloud')
```

---

## requirements.txt

```
streamlit
pandas
numpy
matplotlib
plotly
```

---

## README.md

```markdown
# Análisis Técnico de Soluciones de Almacenamiento de Datos

Proyecto demo: dashboard interactivo en Streamlit que compara HDD, SSD, Cintas y Nube desde criterios técnicos y presenta visualizaciones, simulación y recomendaciones.

## Qué incluye
- `app.py` : aplicación principal en Streamlit
- `requirements.txt` : librerías necesarias
- `assets/infra_diagram.png` : diagrama de infraestructura (subir imagen propia)

## Funcionalidades
- Tabla comparativa técnica (velocidad, capacidad, costo, fiabilidad, consumo, seguridad, escalabilidad)
- Gráficos: barras (velocidad/capacidad), barras (costo/GB), radar (fiabilidad/escalabilidad/seguridad)
- Simulación de crecimiento de datos (50% en 2 años) y comparación de tiempos de restauración HDD vs SSD
- Diagrama de infraestructura estático
- Análisis de riesgos y oportunidades
- Conclusión técnica: recomendación de solución híbrida (SSD + Nube)

## Capturas / GIF
Incluye capturas o GIFs del dashboard en la carpeta `assets/` (opcional).

## Despliegue en Streamlit Cloud
1. Crea un nuevo repositorio en GitHub llamado `data-storage-analysis`.
2. Sube los archivos (`app.py`, `requirements.txt`, `assets/infra_diagram.png`, `README.md`).
3. Ve a https://streamlit.io/cloud y conecta tu cuenta de GitHub.
4. Selecciona el repositorio `data-storage-analysis` y selecciona `app.py` como entrypoint.
5. Despliega y comparte la URL pública.

---

## Notas finales
- Los números en la comparativa son ilustrativos. Para producción, sustituir por medidas/benchmarks reales.
- Asegúrate de subir una imagen `assets/infra_diagram.png` para que el diagrama se muestre en la app.
```

---

## Instrucciones extra

* Sube estos archivos exactamente con los nombres indicados.
* Añade `assets/infra_diagram.png` (un diagrama PNG de la arquitectura híbrida) al repo.

¡Listo! Sube el repo a GitHub y conecta a Streamlit Cloud (ver README) para tener el dashboard corriendo en la web.
