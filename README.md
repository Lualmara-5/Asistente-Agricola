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

### 1. Instalación de dependencias
```bash
!pip install git+https://github.com/nilp0inter/experta.git
!pip install rdflib
!pip install owlrl
!pip install scikit-fuzzy
!pip install matplotlib
!pip install numpy
```

### 2. Ontología y Razonamiento Semántico
- Crear ontología en Turtle con:
      - ≥10 clases (ej: Cultivo, Suelo, Plaga, Fertilizante…).
      - ≥10 propiedades (con `domain` y `range`).
      - Jerarquías (`rdfs:subClassOf`, `rdfs:subPropertyOf`).
      - Individuos (≥4 por clase).
- Aplicar **razonamiento** con `DeductiveClosure(RDFS_Semantics)`.
- Comparar grafo antes y después → mostrar inferencias.

### 3. Lógica Difusa (Scikit-Fuzzy)
- 3 variables difusas (`Humedad`, `Temperatura`, `Radiación`).  
- Universo de discurso y funciones de pertenencia (triangular, trapezoidal, gaussiana). 
- Uso de modificadores (*muy*, *ligeramente*).  
- ≥9 reglas con operadores `AND`, `OR`, `NOT`.  
- Proceso de defuzzificación (centroide).
- Graficar funciones de pertenencia.

### 4. Sistema Experto (Experta)
- Crear ≥5 clases de hechos (ej: `CultivoFact`, `SueloFact`, `PlagaFact`…).
- Base de hechos inicial (≥40 hechos, incluyendo los inferidos de la ontología).
- ≥15 reglas con diferentes prioridades (`salience`) y control de ejecución.
- Reglas que usen resultados de ontología + lógica difusa.
- Evitar conflictos (por ejemplo con control `halt()` o condiciones exclusivas).

### 5. Integración
- Traducir hechos de la ontología (triples inferidos) a hechos en Experta.
- Pasar resultados del difuso (ej: “humedad baja”, “temperatura alta”) como condiciones.
- El sistema experto combina todo y produce recomendaciones agrícolas (ej: *“Activar riego y aplicar fertilizante nitrogenado”*).

---

## 📂 Archivos del Repositorio
```
.
├── Asistente_Agricola.ipynb   # Notebook con todo el desarrollo (experto, difuso, ontología, integración)
├── README.md                  # Este archivo
└── LICENSE                    # Licencia del proyecto (MIT)
```

---

## 📑 Entregables
- `Asistente_Agricola.ipynb` (notebook único con todo el código).  
- `README.md` (instrucciones y documentación).   
- `LICENSE`.  
- Documento PDF de diseño (en Classroom).  
- Video pitch de sustentación (enlace incluido en el PDF).  

---

## 👥 Autores
- Grupo XX – Equipo YY  
- Curso: **Introducción a la Inteligencia Artificial (3010476)**  
- Profesor: Jaime Alberto Guzmán Luna, Ph.D.  
