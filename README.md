# 🌱 Asistente Inteligente para Cultivo Agrícola

## 📌 Descripción General
Este proyecto corresponde a la **Práctica 1** del curso *Introducción a la Inteligencia Artificial (3010476)* de la Universidad Nacional de Colombia.  
El objetivo es implementar un **sistema híbrido de IA** que integre:

- **Sistema experto (Experta – Python):** reglas basadas en hechos para decisiones agrícolas.  
- **Lógica difusa (Scikit-Fuzzy):** modelado de incertidumbre en variables ambientales.  
- **Ontología (RDFLib + OWL-RL):** representación semántica de cultivos, suelos, plagas y fertilizantes.  

El dominio seleccionado es la **agricultura de cultivos básicos**, enfocado en la **gestión de riego y fertilización**.

---

## 🎯 Objetivos
- Representar conocimiento agrícola mediante una ontología formal.  
- Modelar variables inciertas (humedad, temperatura, radiación) con lógica difusa.  
- Integrar razonamiento semántico y difuso en un sistema experto que sugiera acciones concretas (ejemplo: *“Activar riego y aplicar fertilizante nitrogenado”*).  

---

## 🧩 Componentes dentro del Notebook
Todo el desarrollo está contenido en **`Asistente_Agricola.ipynb`**, dividido en secciones:

### 1. Ontología y Razonamiento (RDFLib + OWL-RL)
- ≥10 clases, ≥10 propiedades con `domain` y `range`, ≥5 jerarquías, ≥4 individuos/clase.  
- Inferencias automáticas con **DeductiveClosure(RDFS_Semantics)**.  
- Comparación de grafo antes y después del razonamiento.  

### 2. Lógica Difusa (Scikit-Fuzzy)
- 3 variables difusas (`Humedad`, `Temperatura`, `Radiación`).  
- Funciones de pertenencia: triangular, trapezoidal, gaussiana.  
- Uso de modificadores (*muy*, *ligeramente*).  
- ≥9 reglas con operadores `AND`, `OR`, `NOT`.  
- Defuzzificación con el método del centroide.  

### 3. Sistema Experto (Experta)
- ≥5 clases de hechos y ≥40 hechos iniciales.  
- ≥15 reglas con 3 niveles de prioridad (`salience`).  
- Dos técnicas de control de ejecución y manejo de conflictos.  
- Uso de hechos provenientes de la ontología y resultados difusos.  

### 4. Integración
- Hechos inferidos de la ontología → Sistema experto.  
- Salidas del difuso → condiciones para activar reglas.  
- Motor de inferencia produce recomendaciones agrícolas.  

---

## 📂 Archivos del Repositorio
```
.
├── Asistente_Agricola.ipynb   # Notebook con todo el desarrollo (experto, difuso, ontología, integración)
├── README.md                  # Este archivo
├── requirements.txt           # Librerías necesarias
└── LICENSE                    # Licencia del proyecto
```

---

## ▶️ Ejecución
1. Instalar dependencias:
   ```bash
   pip install experta rdflib owlrl scikit-fuzzy
   ```
2. Abrir y ejecutar el notebook:
   ```bash
   jupyter notebook Asistente_Agricola.ipynb
   ```

---

## 📑 Entregables
- `Asistente_Agricola.ipynb` (notebook único con todo el código).  
- `README.md` (instrucciones y documentación).  
- `requirements.txt` (dependencias).  
- `LICENSE`.  
- Documento PDF de diseño (en Classroom).  
- Video pitch de sustentación (enlace incluido en el PDF).  

---

## 👥 Autores
- Grupo XX – Equipo YY  
- Curso: **Introducción a la Inteligencia Artificial (3010476)**  
- Profesor: Jaime Alberto Guzmán Luna, Ph.D.  
