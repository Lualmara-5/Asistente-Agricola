# Asistente Agrícola - Explicación del Código

Este documento explica paso a paso la implementación del proyecto **Asistente Agrícola**, desarrollado en Python sobre un entorno de Jupyter Notebook (Google Colab).  

El flujo del sistema está diseñado de la siguiente manera:  
**Ontología → Difusa → Experto → Integración final**

---

## 1. Introducción

El proyecto busca implementar un **asistente agrícola inteligente** que apoye la toma de decisiones de los agricultores mediante tres componentes principales:

1. **🧩 Ontología**  
   - Representa el conocimiento del dominio agrícola.  
   - Organiza conceptos como tipos de cultivos, plagas, climas y tratamientos.  
   - Permite establecer relaciones y hacer consultas sobre la base de conocimiento.

2. **🌫️ Lógica Difusa**  
   - Permite manejar información imprecisa (ejemplo: “clima cálido”, “suelo húmedo”).  
   - Se utilizan funciones de pertenencia para transformar valores numéricos en categorías lingüísticas.  
   - Facilita tomar decisiones en escenarios donde no todo es 0 o 1.

3. **🤖 Sistema Experto**  
   - Contiene reglas de producción tipo “SI … ENTONCES …”.  
   - Usa la información proporcionada por la ontología y el módulo difuso para inferir recomendaciones.  
   - Genera las salidas finales (ejemplo: sugerencia de cultivo, tratamiento o acción preventiva).

En conjunto, el asistente agrícola integra estos tres componentes para dar una **recomendación final coherente**.


---

## 2. Estructura del Código

El código está implementado en un **Notebook de Google Colab**, lo que facilita la ejecución en la nube sin necesidad de instalar manualmente un entorno local.  

---

## 3. Explicación por Módulos

### 📦 Instalar Dependencias
```python
!pip install git+https://github.com/nilp0inter/experta.git
!pip install rdflib
!pip install owlrl
!pip install scikit-fuzzy
!pip install matplotlib
!pip install numpy
```

#### Explicación:
Estas líneas instalan las librerías necesarias para los tres módulos del asistente:
- `experta`: motor de reglas para implementar el sistema experto.
- `rdflib`: librería para trabajar con grafos RDF y construir la ontología.
- `owlrl`: permite aplicar razonamiento RDFS/OWL y generar inferencias automáticas.
- `scikit-fuzzy`: librería para lógica difusa (funciones de pertenencia y control difuso).
- `matplotlib`: para graficar resultados de las funciones difusas.
- `numpy`: para operaciones matemáticas.

---

### 📚 Importación de librerías principales
```python
# Ontologías
from rdflib import Graph, Namespace, RDF, RDFS, XSD, Literal, URIRef
from owlrl import DeductiveClosure, RDFS_Semantics

# Lógica Difusa
import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# Sistema Experto
from experta import *
```

#### Explicación:
En esta sección se importan los módulos clave que usaremos:

- 🧩 Ontologías:
    - `Graph`: estructura principal para almacenar nodos y relaciones RDF.
    - `Namespace`: define espacios de nombres (ej: AGRI: para conceptos agrícolas).
    - `RDF`, `RDFS`, `XSD`: vocabularios estándar de la web semántica.
    - `Literal`, `URIRef`: permiten crear valores y referencias únicas dentro del grafo.
    - `DeductiveClosure`, `RDFS_Semantics`: aplican razonamiento automático sobre el grafo.

- 🌫️ Lógica Difusa:
    - `numpy:` cálculos matemáticos y arreglos.
    - `skfuzzy`: permite crear funciones de pertenencia (ej: humedad baja/media/alta).
    - `control`: módulo de sistemas difusos completos (entradas, reglas, salidas).
    - `matplotlib`: visualización de funciones y resultados.

- 🤖 Sistema Experto:

    - `experta`: motor de inferencia basado en reglas (hereda de `pyknow`). <br> Con ella se definen hechos y reglas que guían las recomendaciones.

---

### 🧩 Ontología y Razonamiento Semántico
En esta sección vamos a definir las clases, propiedades, individuos y aplicaremos razonamiento.

#### 1. Crear Grafo y Namespaces
```python
g = Graph()
EX = Namespace("http://example.org/agricola#")
FOAF = Namespace("http://xmlns.com/foaf/0.1/")
DC = Namespace("http://purl.org/dc/elements/1.1/")

g.bind("ex", EX)
g.bind("foaf", FOAF)
g.bind("dc", DC)
g.bind("rdfs", RDFS)
g.bind("rdf", RDF)
g.bind("xsd", XSD)
```

#### Explicación:
- Se crea un grago RDF vacío (contenedor de triples: sujeto-predicado-objeto). Todo lo que modelemos (clases, propiedades, individuos) van a ir aquí (Ejemplo: (Maiz, tipo, Cultivo)).
- `Namespace(...)` crea prefijos reutilizables para URIs.
    - `EX` es tu espacio de nombres del proyecto (aquí `http://example.org/agricola#`). Cada recurso se puede referenciar como `EX.Cultivo`, `EX.Maiz`, etc.
    - `FOAF` y `DC` son vocabularios estándar: `FOAF` es útil si modelas personas (agricultores), `DC` para metadatos (título, autor). Usar vocabularios estándar mejora interoperabilidad.
- `g.bind(prefix, namespace)` asocia el prefijo para que, al serializar el grafo (por ejemplo a Turtle), se vea `ex:Maiz` en vez de la URI completa — hace la salida humana y legible.
- También enlazaste `rdfs`, `rdf` y `xsd` para que aparezcan como prefijos en la serialización y para usar vocabulario estándar (`RDFS.Class`, `rdf:type`, `xsd:float`, ...).

#### Explicación no tan tecnica:
- `EX` → tu cajita de conceptos inventados (Cultivo, Plaga, Suelo).
- `FOAF` → cajita estándar para “persona, nombre, organización”.
- `DC` → cajita estándar para “metadatos” (autor, título, fecha).
- Sin bind:
```bash
<http://example.org/agricola#Maiz> a <http://example.org/agricola#Cultivo> .
```
- Con bind:
```bash
ex:Maiz a ex:Cultivo .
```

---

#### 2. Definir Clases (>=10)
```python
class_definitions = {
    "AgriculturalEntity": EX.AgriculturalEntity,
    "Organismo": EX.Organismo,
    "Cultivo": EX.Cultivo,
    "Plaga": EX.Plaga,
    "Beneficio": EX.Beneficio,
    "Suelo": EX.Suelo,
    "Fertilizante": EX.Fertilizante,
    "SistemaRiego": EX.SistemaRiego,
    "Clima": EX.Clima,
    "Nutriente": EX.Nutriente,
    "PracticaAgricola": EX.PracticaAgricola,
    "Enfermedad": EX.Enfermedad
}

for class_name, class_uri in class_definitions.items():
    g.add((class_uri, RDF.type, RDFS.Class))
    g.add((class_uri, RDFS.label, Literal(class_name)))
```
#### Explicación:

1. **`class_definitions`** contiene un mapeo de nombres legibles a URIs en el namespace `EX`. Esto te permite definir muchas clases de forma compacta.
2. Cada entrada en el bucle añade dos triples al grafo:

   * `(class_uri, RDF.type, RDFS.Class)`: declara que el recurso es una clase RDF (clase formal dentro de la ontología).
   * `(class_uri, RDFS.label, Literal(class_name))`: proporciona una etiqueta legible (útil en visualizadores y para documentación).

**Descripción breve de las clases:**

* `AgriculturalEntity`: clase raíz que agrupa entidades agrícolas.
* `Organismo`: seres vivos (plagas, microorganismos beneficiosos).
* `Cultivo`: tipos de cultivo (maíz, arroz, etc.).
* `Plaga`: insectos u organismos nocivos.
* `Beneficio`: elementos que benefician el cultivo (polinizadores, control biológico).
* `Suelo`: tipos y propiedades del suelo.
* `Fertilizante`: insumos aplicables al suelo/plantas.
* `SistemaRiego`: métodos de riego (goteo, aspersión).
* `Clima`: condiciones ambientales como lluvia, temperatura.
* `Nutriente`: N, P, K y otros nutrientes.
* `PracticaAgricola`: rotación, labranza, uso de coberturas.
* `Enfermedad`: enfermedades que afectan los cultivos.

**Resultado esperado (Turtle - ejemplo):**

```turtle
ex:Cultivo a rdfs:Class ;
    rdfs:label "Cultivo" .
```

---

#### 3. Jerarquías (>=5 subClassOf)
```python
# (Cultivo -> Organismo -> AgriculturalEntity), etc.
g.add((EX.Organismo, RDFS.subClassOf, EX.AgriculturalEntity))
g.add((EX.Cultivo, RDFS.subClassOf, EX.Organismo))
g.add((EX.Plaga, RDFS.subClassOf, EX.Organismo))
g.add((EX.Beneficio, RDFS.subClassOf, EX.Organismo))
g.add((EX.Suelo, RDFS.subClassOf, EX.AgriculturalEntity))
g.add((EX.PracticaAgricola, RDFS.subClassOf, EX.AgriculturalEntity))
g.add((EX.SistemaRiego, RDFS.subClassOf, EX.PracticaAgricola))
g.add((EX.Enfermedad, RDFS.subClassOf, EX.Organismo))
```
#### Explicación:

* `rdfs:subClassOf`: indica que una clase es una **subclase** de otra, heredando sus propiedades.
* Ejemplos:

  * `ex.Cultivo rdfs:subClassOf ex.Organismo`: todo cultivo es un tipo de organismo.
  * `ex.Organismo rdfs:subClassOf ex.AgriculturalEntity`: cualquier organismo pertenece al dominio agrícola.
  * `ex.SistemaRiego rdfs:subClassOf ex.PracticaAgricola`: el riego es una práctica agrícola.
* Esto crea una **jerarquía taxonómica** que permite razonamiento: si algo es un `Cultivo`, automáticamente es también un `Organismo` y una `AgriculturalEntity`.

---

#### 4. Definir Propiedades (>=10) con domain y range
```python
property_definitions = {
    "seCultivaEn": (EX.seCultivaEn, EX.Cultivo, EX.Suelo),
    "tienePlaga": (EX.tienePlaga, EX.Cultivo, EX.Plaga),
    "requiereFertilizacion": (EX.requiereFertilizacion, EX.Cultivo, EX.Fertilizante),
    "tienePH": (EX.tienePH, EX.Suelo, XSD.decimal),
    "tieneHumedad": (EX.tieneHumedad, EX.Suelo, XSD.decimal),
    "necesitaRiego": (EX.necesitaRiego, EX.Cultivo, XSD.boolean),
    "tieneRendimiento": (EX.tieneRendimiento, EX.Cultivo, XSD.decimal),
    "tiempoCosecha": (EX.tiempoCosecha, EX.Cultivo, XSD.integer),
    "afecta": (EX.afecta, EX.Plaga, EX.Cultivo),
    "reduceRendimiento": (EX.reduceRendimiento, EX.Plaga, EX.Cultivo),
    "contieneNutriente": (EX.contieneNutriente, EX.Fertilizante, EX.Nutriente),
    "recomendadoPara": (EX.recomendadoPara, EX.Fertilizante, EX.Cultivo),
    "esCompatibleCon": (EX.esCompatibleCon, EX.SistemaRiego, EX.Cultivo),
    "tieneMetodo": (EX.tieneMetodo, EX.PracticaAgricola, EX.SistemaRiego)
}

for property_name, (property_uri, property_domain, property_range) in property_definitions.items():
    g.add((property_uri, RDF.type, RDF.Property))
    g.add((property_uri, RDFS.label, Literal(property_name)))
    g.add((property_uri, RDFS.domain, property_domain))
    # si range es un XSD type (URIRef), lo ponemos tal cual
    g.add((property_uri, RDFS.range, property_range))

# subPropertyOf: reduceRendimiento subPropertyOf afecta
g.add((EX.reduceRendimiento, RDFS.subPropertyOf, EX.afecta))
```
#### Explicación:

En esta parte definimos las **propiedades** de la ontología, es decir, las relaciones o atributos que pueden tener las clases e individuos.  

- **`rdf:Property`**: declara que el recurso es una propiedad.  
- **`rdfs:label`**: nombre legible de la propiedad.  
- **`rdfs:domain`**: indica desde dónde se aplica la propiedad (la clase de origen).  
- **`rdfs:range`**: indica hacia dónde apunta la propiedad (la clase o tipo de dato destino).  

Ejemplos concretos:  
- `ex:seCultivaEn` → **domain**: `ex:Cultivo`, **range**: `ex:Suelo`  
  > Un cultivo se cultiva en un suelo.  
- `ex:tienePlaga` → **domain**: `ex:Cultivo`, **range**: `ex:Plaga`  
  > Un cultivo puede estar afectado por una plaga.  
- `ex:tienePH` → **domain**: `ex:Suelo`, **range**: `xsd:decimal`  
  > Un suelo puede tener un valor decimal como pH.  

Además, se define una **jerarquía de propiedades**:  
- `ex:reduceRendimiento rdfs:subPropertyOf ex:afecta`  
  > Si una plaga reduce el rendimiento de un cultivo, automáticamente también lo afecta.  

En conjunto, estas propiedades permiten describir **atributos** (ej. pH, humedad) y **relaciones** (ej. un cultivo tiene plaga, un fertilizante contiene nutriente) dentro del dominio agrícola.  

---

#### 5. Instanciar individuos (>=4 individuos por clase principal)
```python
```
#### Explicación:

---

#### 6. Añadir algunas relaciones entre individuos (hechos) para que el razonador pueda inferir
```python
```
#### Explicación:

---

#### 7. Guardar estado previo del razonamiento (para comparar)
```python
```
#### Explicación:

---

#### 8. Aplicar razonamiento RDFS con OWL-RL (DeductiveClosure)
```python
```
#### Explicación:

---

#### 9. Comparar y mostrar inferencias nuevas
```python
```
#### Explicación:

---

#### 10. Serializar grafo final (post-razonador) en Turtle
```python
```
#### Explicación:

---

#### 11. Función para buscar triples por patrón
```python
```
#### Explicación:

---

### 🌫️ Lógica Difusa: Humedad, Temperatura, Radiación
### 🤖 Sistema Experto (Experta)

---

## 4. Integración de los Módulos
- Cómo se comunican entre sí.
- Flujo del programa.

---

## 5. Ejecución de Pruebas
- Ejemplo de entradas y salidas.
- Casos de uso probados.

---

## 6. Conclusiones
- Reflexión sobre lo aprendido.
- Posibles mejoras.

---
