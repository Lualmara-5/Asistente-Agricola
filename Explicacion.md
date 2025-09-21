# 🌱 Asistente Agrícola - Explicación del Código

Este documento explica paso a paso la implementación del proyecto **Asistente Agrícola**, desarrollado en Python sobre un entorno de Jupyter Notebook (Google Colab).  

El flujo del sistema está diseñado de la siguiente manera:  
**Ontología → Difusa → Experto → Integración final**

---

## 📌 1. Introducción

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
# Cultivos
cultivos = {
    "Maiz": EX.Maiz,
    "Trigo": EX.Trigo,
    "Arroz": EX.Arroz,
    "Frijol": EX.Frijol
}
for name, uri in cultivos.items():
    g.add((uri, RDF.type, EX.Cultivo))
    g.add((uri, RDFS.label, Literal(name)))
    g.add((uri, FOAF.name, Literal(name)))

# Suelos
suelos = {
    "SueloArcilloso": EX.SueloArcilloso,
    "SueloArenoso": EX.SueloArenoso,
    "SueloLimoso": EX.SueloLimoso,
    "SueloFranco": EX.SueloFranco
}
for name, uri in suelos.items():
    g.add((uri, RDF.type, EX.Suelo))
    g.add((uri, RDFS.label, Literal(name)))

# Plagas
plagas = {
    "Langosta": EX.Langosta,
    "GusanoCogollero": EX.GusanoCogollero,
    "MoscaBlanca": EX.MoscaBlanca,
    "Roya": EX.Roya
}
for name, uri in plagas.items():
    g.add((uri, RDF.type, EX.Plaga))
    g.add((uri, RDFS.label, Literal(name)))

# Fertilizantes
fertilizantes = {
    "Nitrogenado": EX.Nitrogenado,
    "Fosfatado": EX.Fosfatado,
    "Potasico": EX.Potasico,
    "Compost": EX.Compost
}
for name, uri in fertilizantes.items():
    g.add((uri, RDF.type, EX.Fertilizante))
    g.add((uri, RDFS.label, Literal(name)))

# SistemaRiego
sistemas = {
    "RiegoPorGoteo": EX.RiegoPorGoteo,
    "RiegoPorAspersion": EX.RiegoPorAspersion,
    "RiegoManual": EX.RiegoManual,
    "RiegoSubterraneo": EX.RiegoSubterraneo
}
for name, uri in sistemas.items():
    g.add((uri, RDF.type, EX.SistemaRiego))
    g.add((uri, RDFS.label, Literal(name)))

# Nutrientes
nutrientes = {
    "Nitrogeno": EX.Nitrogeno,
    "Fosforo": EX.Fosforo,
    "Potasio": EX.Potasio,
    "Calcio": EX.Calcio
}
for name, uri in nutrientes.items():
    g.add((uri, RDF.type, EX.Nutriente))
    g.add((uri, RDFS.label, Literal(name)))

# PracticasAgricolas
practicas = {
    "Labranza": EX.Labranza,
    "SiembraDirecta": EX.SiembraDirecta,
    "RotacionCultivos": EX.RotacionCultivos,
    "CoberturaSuelo": EX.CoberturaSuelo
}
for name, uri in practicas.items():
    g.add((uri, RDF.type, EX.PracticaAgricola))
    g.add((uri, RDFS.label, Literal(name)))

# Climas
climas = {
    "Tropical": EX.Tropical,
    "Seco": EX.Seco,
    "Humedo": EX.Humedo,
    "Templado": EX.Templado
}
for name, uri in climas.items():
    g.add((uri, RDF.type, EX.Clima))
    g.add((uri, RDFS.label, Literal(name)))

# Beneficios (organismos beneficiosos)
beneficios = {
    "Abeja": EX.Abeja,
    "Mariquita": EX.Mariquita,
    "Lombriz": EX.Lombriz,
    "BacillusSubtilis": EX.BacillusSubtilis
}
for name, uri in beneficios.items():
    g.add((uri, RDF.type, EX.Beneficio))
    g.add((uri, RDFS.label, Literal(name)))

# Enfermedades
enfermedades = {
    "Mildiu": EX.Mildiu,
    "Rizoctonia": EX.Rizoctonia,
    "VirusMozaico": EX.VirusMozaico,
    "PudricionRizal": EX.PudricionRizal
}
for name, uri in enfermedades.items():
    g.add((uri, RDF.type, EX.Enfermedad))
    g.add((uri, RDFS.label, Literal(name)))
```
#### Explicación

En este bloque se crean **individuos** (ejemplares concretos) para cada clase principal de la ontología, como cultivos, suelos, plagas, fertilizantes, sistemas de riego, nutrientes, prácticas agrícolas, climas, beneficios y enfermedades.

Cada individuo se añade con:
- **`rdf:type`** → indica a qué clase pertenece (ej. `ex:Maiz rdf:type ex:Cultivo`).
- **`rdfs:label`** → un nombre legible para humanos.
- **`foaf:name`** (solo en cultivos) → se usa una propiedad estándar de FOAF para almacenar el nombre de un individuo de forma semántica y reutilizable en otras ontologías.

Así, por ejemplo:
- `ex:Maiz` queda definido como un individuo de la clase `ex:Cultivo`.  
- `ex:SueloArcilloso` es un individuo de la clase `ex:Suelo`.  
- `ex:Langosta` es un individuo de la clase `ex:Plaga`.  

Este paso permite poblar la ontología con ejemplos reales que luego serán utilizados por el razonador para inferir nuevo conocimiento.  

---

#### 6. Añadir algunas relaciones entre individuos (hechos) para que el razonador pueda inferir

En este bloque se agregan **triples concretos** (hechos) que relacionan individuos entre sí o con valores literales. 

Esto permite que el razonador use las **propiedades con dominio y rango** definidas antes para **inferir nuevo conocimiento** automáticamente.

```python
# Maíz se cultiva en SueloFranco (subject ya es tipo Cultivo -> inferencias por domain/range)
g.add((EX.Maiz, EX.seCultivaEn, EX.SueloFranco))
g.add((EX.Trigo, EX.seCultivaEn, EX.SueloLimoso))
g.add((EX.Arroz, EX.seCultivaEn, EX.SueloArcilloso))
g.add((EX.Frijol, EX.seCultivaEn, EX.SueloArenoso))

# Plagas sobre cultivos (y usamos reduceRendimiento que es subPropertyOf afecta)
g.add((EX.Langosta, EX.reduceRendimiento, EX.Maiz))
g.add((EX.GusanoCogollero, EX.reduceRendimiento, EX.Trigo))
g.add((EX.MoscaBlanca, EX.reduceRendimiento, EX.Frijol))
g.add((EX.Roya, EX.reduceRendimiento, EX.Arroz))

# Fertilizantes contienen nutrientes y recomendados
g.add((EX.Nitrogenado, EX.contieneNutriente, EX.Nitrogeno))
g.add((EX.Fosfatado, EX.contieneNutriente, EX.Fosforo))
g.add((EX.Potasico, EX.contieneNutriente, EX.Potasio))
g.add((EX.Compost, EX.contieneNutriente, EX.Calcio))

g.add((EX.Nitrogenado, EX.recomendadoPara, EX.Maiz))
g.add((EX.Compost, EX.recomendadoPara, EX.Frijol))
g.add((EX.Potasico, EX.recomendadoPara, EX.Trigo))

# Sistema de riego compatible con cultivos
g.add((EX.RiegoPorGoteo, EX.esCompatibleCon, EX.Maiz))
g.add((EX.RiegoPorAspersion, EX.esCompatibleCon, EX.Trigo))
g.add((EX.RiegoManual, EX.esCompatibleCon, EX.Frijol))

# Propiedades literales: pH y humedad de suelos (typed literals)
g.add((EX.SueloFranco, EX.tienePH, Literal("6.5", datatype=XSD.decimal)))
g.add((EX.SueloArcilloso, EX.tienePH, Literal("5.8", datatype=XSD.decimal)))
g.add((EX.SueloArenoso, EX.tienePH, Literal("6.8", datatype=XSD.decimal)))
g.add((EX.SueloLimoso, EX.tienePH, Literal("6.0", datatype=XSD.decimal)))

g.add((EX.SueloFranco, EX.tieneHumedad, Literal("0.35", datatype=XSD.decimal)))
g.add((EX.SueloArcilloso, EX.tieneHumedad, Literal("0.45", datatype=XSD.decimal)))
g.add((EX.SueloArenoso, EX.tieneHumedad, Literal("0.20", datatype=XSD.decimal)))
g.add((EX.SueloLimoso, EX.tieneHumedad, Literal("0.30", datatype=XSD.decimal)))

# tiempo de cosecha (ejemplo de literal integer)
g.add((EX.Maiz, EX.tiempoCosecha, Literal("120", datatype=XSD.integer)))
g.add((EX.Trigo, EX.tiempoCosecha, Literal("110", datatype=XSD.integer)))
```
#### Explicación:

- **Cultivo → Suelo**  
  ```ttl
  ex:Maiz ex:seCultivaEn ex:SueloFranco .
  ```
  Como `seCultivaEn` tiene dominio `ex:Cultivo` y rango `ex:Suelo`, el razonador puede verificar que:
  - `ex:Maiz` efectivamente es un `ex:Cultivo`.  
  - `ex:SueloFranco` es un `ex:Suelo`.

- **Plagas que afectan cultivos**  
  Se usa la propiedad `reduceRendimiento` (subPropertyOf `afecta`):
  ```ttl
  ex:Langosta ex:reduceRendimiento ex:Maiz .
  ```
  Esto implica que `Langosta` también **afecta** al `Maiz`, gracias a la jerarquía de propiedades.

- **Fertilizantes y nutrientes**  
  ```ttl
  ex:Nitrogenado ex:contieneNutriente ex:Nitrogeno .
  ex:Nitrogenado ex:recomendadoPara ex:Maiz .
  ```
  El razonador puede concluir que el `ex:Nitrogenado` es útil en cultivos de `ex:Maiz`.

- **Compatibilidad con sistemas de riego**  
  ```ttl
  ex:RiegoPorGoteo ex:esCompatibleCon ex:Maiz .
  ```

- **Propiedades literales de suelos**  
  Se añaden valores tipados (`xsd:decimal`, `xsd:integer`) para pH, humedad y tiempo de cosecha:
  ```ttl
  ex:SueloFranco ex:tienePH "6.5"^^xsd:decimal .
  ex:Maiz ex:tiempoCosecha "120"^^xsd:integer .
  ```
  
---

#### 7. Guardar estado previo del razonamiento (para comparar)

En esta parte lo que hacemos es **guardar un “snapshot” del grafo antes de aplicar el razonador**.  

Esto sirve para después comparar cuántos triples había originalmente y cuántos se generaron tras las inferencias.

```python
before_triples = set(g)
print("Triples antes del razonador:", len(before_triples))

# Serializar la ontología previa (Turtle)
g.serialize(destination="ontologia_pre_razonador.ttl", format="turtle")
print("Ontología previa serializada: ontologia_pre_razonador.ttl")
```
#### Explicación:

- `before_triples = set(g)`  
  Convierte el grafo en un conjunto de triples.  
  Así podemos contar fácilmente cuántos había (`len(before_triples)`).

- `g.serialize(destination="ontologia_pre_razonador.ttl", format="turtle")`  
  Serializa el grafo en formato **Turtle** y lo guarda en un archivo llamado `ontologia_pre_razonador.ttl`.  
  Este archivo contiene solo lo que hemos definido manualmente (sin razonamiento aún).

Ejemplo de salida esperada:
```txt
Triples antes del razonador: 201
Ontología previa serializada: ontologia_pre_razonador.ttl
```

👉 Así, cuando corramos el razonador en el siguiente paso, podremos medir cuántos triples nuevos se infirieron.

---

#### 8. Aplicar razonamiento RDFS con OWL-RL (DeductiveClosure)
```python
DeductiveClosure(RDFS_Semantics).expand(g)
```
#### Explicación:

- **Qué hace:** aplica las reglas de inferencia RDFS/OWL-RL sobre el grafo `g` y **añade triples inferidos** automáticamente (por ejemplo: transitividad de `rdfs:subClassOf`, herencia por `rdfs:subPropertyOf`, inferencias a partir de `rdfs:domain`/`rdfs:range`, etc.).

- **Efecto en el grafo:** modifica `g` **in-place**; después de ejecutar `expand(g)` el grafo contendrá tanto los triples originales como los nuevos triples inferidos.

- **Ejemplos de inferencias típicas:**
  - Si `ex:reduceRendimiento rdfs:subPropertyOf ex:afecta` y existe `ex:Langosta ex:reduceRendimiento ex:Maiz` → el razonador infiere `ex:Langosta ex:afecta ex:Maiz`.
  - Si `ex:seCultivaEn rdfs:domain ex:Cultivo` y existe `ex:Maiz ex:seCultivaEn ex:SueloFranco` → el razonador puede inferir `ex:Maiz rdf:type ex:Cultivo` (si no estaba explícito).

---

#### 9. Comparar y mostrar inferencias nuevas
```python
after_triples = set(g)
print("Triples después del razonador:", len(after_triples))

inferred = after_triples - before_triples
print("Nuevos triples inferidos por el razonador (ejemplos, hasta 40):", len(inferred))
# Mostrar hasta 40 triples inferidos para inspección
count = 0
for s,p,o in inferred:
    # imprimimos en forma legible
    print(f"- {s}  {p}  {o}")
    count += 1
    if count >= 40:
        break

# Verificar al menos 4 hechos inferidos (condición ejercicio)
if len(inferred) >= 4:
    print("✅ Se infirieron al menos 4 hechos nuevos.")
else:
    print("⚠️ No se detectaron 4 hechos inferidos (revisar la ontología).")
```
#### Explicación:

- `after_triples = set(g)` → guarda el conjunto de triples **después del razonador**.
- `inferred = after_triples - before_triples` → resta conjuntos: deja **solo los triples nuevos** que antes no existían.
- El bucle `for s,p,o in inferred:` → recorre cada nuevo triple (sujeto, predicado, objeto).
- `count >= 40` → corta la impresión si hay demasiados, mostrando solo los primeros 40.
- El `if len(inferred) >= 4` → comprueba la condición del ejercicio: confirmar al menos 4 inferencias nuevas.

---

#### 10. Serializar grafo final (post-razonador) en Turtle
```python
g.serialize(destination="ontologia_post_razonador.ttl", format="turtle")
print("Ontología post-razonador serializada: ontologia_post_razonador.ttl")
```
#### Explicación:

- `g.serialize(...)` → convierte el grafo `g` en texto Turtle y lo escribe en un archivo.
  - `destination="ontologia_post_razonador.ttl"` → nombre del archivo de salida.
  - `format="turtle" `→ formato de serialización.

- Este grafo ya contiene:
1. Los **triples base** que definimos manualmente.
2. Los **triples inferidos** automáticamente por el razonador.
- La impresión en consola confirma que el archivo se generó.

---

#### 11. Función para buscar triples por patrón
```python
def buscar(s=None,p=None,o=None, limit=20):
    results = list(g.triples((s,p,o)))
    print(f"Resultados: {len(results)} (mostrando hasta {limit})")
    for i, (ss,pp,oo) in enumerate(results):
        if i >= limit:
            break
        print(ss, pp, oo)

# Ejemplo: buscar todo lo relacionado con Maiz
print("\n--- Ejemplo: triples (Maiz, ?, ?) ---")
buscar(EX.Maiz, None, None, limit=40)
```
#### Explicación:

- **La función** `buscar` permite consultar triples dentro del grafo RDF.
- **Parámetros opcionales** (`s`, `p`, `o`):
   - Si se especifican, filtran por sujeto, predicado u objeto.
   - Si se deja en `None`, no se filtra por ese campo.
- **Consulta de triples**: `g.triples((s,p,o))` devuelve todos los que cumplen el patrón.
- **Control de salida (`limit`)**: restringe la cantidad de resultados mostrados.
- **Ejemplo con `EX.Maiz`**: imprime todas las propiedades y relaciones en las que aparece el *individuo Maíz*.

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
