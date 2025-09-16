# 🌱 Asistente Agrícola: Sistema Híbrido de IA para Cultivo Agrícola

Este proyecto implementa un **sistema híbrido de inteligencia artificial** que combina **Ontologías (RDF/OWL-RL)**, **Lógica Difusa (Scikit-Fuzzy)** y **Sistemas Expertos (Experta)** para apoyar la toma de decisiones en el cuidado de cultivos agrícolas.

---

## 📌 Introducción al problema
Los agricultores enfrentan condiciones inciertas como **humedad del suelo, temperatura ambiental y presencia de plagas**.  
Este sistema busca ofrecer **recomendaciones inteligentes** integrando tres paradigmas de IA clásica:

- **Ontologías** → Para representar el conocimiento agrícola (cultivos, suelos, plagas, fertilizantes).  
- **Lógica Difusa** → Para modelar incertidumbre en variables del entorno (humedad, temperatura, radiación solar).  
- **Sistema Experto** → Para inferir acciones recomendadas (riego, fertilización, control de plagas) a partir de hechos y reglas.  

---

## 🎯 Objetivos
- Diseñar una **ontología agrícola** con entidades, relaciones e individuos.  
- Modelar variables ambientales con **lógica difusa** y generar salidas interpretables.  
- Implementar un **sistema experto basado en reglas** que integre hechos de la ontología y condiciones difusas.  
- Ejecutar un **caso de prueba completo** que muestre recomendaciones agrícolas en escenarios realistas.  

---

## 📂 Organización del proyecto
```plaintext
Asistente-Agricola/
│
├── README.md          # Guía del proyecto y checklist del equipo
├── requirements.txt   # Dependencias del proyecto
│
├── ontology/          # Ontología en RDF/OWL-RL
│   ├── agro.ttl
│   ├── build_ontology.py
│   ├── reason.py
│   └── mapping.py
│
├── fuzzy/             # Modelo difuso
│   ├── fuzzy_model.py
│   └── plots/*.png
│
├── expert/            # Sistema experto (Experta)
│   ├── hechos.py
│   ├── reglas.py
│   ├── engine.py
│   └── integracion.py
│
├── app.py             # Script principal de integración
└── docs/              # Documentación y entregables
    └── diseño.pdf
```

---

## 🛠️ Configuración de entorno virtual

### 🔹 Opción 1: con `venv` (Python estándar)
1. Crear entorno virtual:
   ```bash
   python -m venv .venv
   ```
2. Activar entorno virtual:  
   - En Linux/Mac:
     ```bash
     source .venv/bin/activate
     ```
   - En Windows:
     ```bash
     .venv\Scripts\activate
     ```
3. Instalar dependencias:
   ```bash
   pip install -r requirements.txt
   ```

### 🔹 Opción 2: con `conda`
1. Crear entorno:
   ```bash
   conda create --name asistente_agricola python=3.10
   ```
2. Activar entorno:
   ```bash
   conda activate asistente_agricola
   ```
3. Instalar dependencias:
   ```bash
   pip install -r requirements.txt
   ```

---

## ✅ Dependencias principales (`requirements.txt`)
```txt
experta
rdflib
owlrl
scikit-fuzzy
matplotlib
```

---

## 🚀 Flujo de Ejecución
1. Construir y razonar ontología:
   ```bash
   python ontology/build_ontology.py
   python ontology/reason.py
   ```

2. Ejecutar sistema difuso:
   ```bash
   python fuzzy/fuzzy_model.py
   ```

3. Probar sistema experto:
   ```bash
   python expert/engine.py
   ```

4. Integración completa (pipeline):
   ```bash
   python app.py
   ```

---

## 👨‍💻 Equipo de desarrollo
- *Luis Alejandro Martínez Ramírez*  
- *Juan Felipe Miranda*  
- *Daniel Felipe Garzón Acosta*  
- *Harrison [Apellido]*  

📅 **Entrega**: 23 de septiembre, 9:50 am  
