# Tarea 1. Minería de textos

# Jorge Pablo Ávila Gómez

# 1. La tarea CoNLL-2002

## 1.1 Descripción de la tarea CoNLL-2002

El principal objetivo de la tarea CoNLL-2002 era el desarrollo de sistemas de reconocimiento de entidades nombradas, que incluyesen componentes de aprendizaje automático. En especial se buscaba el desarrollo de sistemas NER, que sean independientes del lenguaje empleado. En esta tarea se estudiaron textos en dos idiomas, español y holandés. Otro punto importante es que los organizadores animaban a que los participantes utilizaran textos adicionales sin etiquetar para mejorar los resultados.

La tarea se basaba en clasificar 4 tipos de entidades:

- PER: personas
- ORG: Organizaciones
- LOC: Localizaciones
- MISC: Misceláneos

Los datos que se ofrecían para la tarea estaban divididos en tres grupos, datos de entrenamiento, de desarrollo, y de test. El objetivo era usar los datos de entrenamiento para ajustar el modelo, los datos de desarrollo para ajustar los hiperparámetros, y los datos de test para evaluar el modelo.

Los datos están divididos por líneas, con una palabra en cada línea seguida de una etiqueta que indica la entidad nombrada a la que pertenece o la etiqueta *O* si no pertenece a ninguna. Además, se utilizan los prefijos *B-,* para indicar si es la primera palabra de la entidad nombrada, o *I-*, para el resto de palabras de la entidad nombrada. En general, este tipo de esquema de etiquetado es una variante del esquema IOB.

Los datos en español corresponden a noticias de la agencia EFE. El archivo que vamos a usar en este trabajo es el conjunto de test, *esp.testb,* con 53049 líneas. 

Los datos se evaluaron midiendo la precisión, cobertura (recall) y medida-F. Una entidad nombrada solo se considera correcta si la predicción es exactamente la etiqueta correspondiente en los datos de test.

Link para descargar el artículo y los datos:

[paper](https://www.aclweb.org/anthology/W02-2024.pdf)

[task description](https://www.clips.uantwerpen.be/conll2002/pdf/15558tjo.sh.pdf)

[files](https://www.clips.uantwerpen.be/conll2002/ner/data/)

## 1.2 Resultados

Participaron un total de 12 grupos. Los cuales usaron diferentes algoritmos de aprendizaje como boosted trees, decision trees, hidden Markov models, maximum entropy models, memory based methods, support vector machines y transformation-based methods.

Los resultados de los diferentes equipos se compararon con un modelo base que solo seleccionaba entidades nombradas que aparecían en los datos de entrenamiento. Es decir, el objetivo del modelo base era tener una precisión alta. Sin embargo, el modelo base presentó una precisión menor que todos o casi todos los equipos. Al parecer debido a errores en el etiquetado en los datos de entrenamiento.

Dentro de las diferentes estrategias que utilizaron los equipos para mejorar el aprendizaje automático, podemos resaltar algunas de ellas como: información ortográfica, boosting, delimitación de las entidades y el uso de entidades externas anotadas. Por otro lado el uso de entidades externas no anotadas no mejoro significativamente los algoritmos en general. 

Globalmente, todos los modelos consiguieron mejorar los resultados del modelo base. De entre todos los participantes, el grupo de Carreras et al. fue el que consiguió mejores resultados para ambos idiomas, con una puntuación F de 81,39 para español y 77,05 para holandés. El modelo que este grupo utilizó estaba basado en boosted decision trees. 

# 2. Objetivos del trabajo

Los objetivos del trabajo son utilizar un etiquetador de entidades nombradas para predecir las etiquetas del texto en español *esp.testb,* usando un modelo entrenado. Evaluar los resultados obtenidos, y estudiar las causas de error y posibles mejoras.

# 3. Herramientas utilizadas

Para este trabajo se va a etiquetar el texto *esp.testb* de la competición CoNLL-2002. 

Para la tarea se ha usado el módulo `spaCy` de Python. `spaCy` puede reconocer diferentes tipos de entidades nombradas en un documento. Para ello necesita un modelo entrenado, el cual se usará para predecir las entidades. Para documentos en español `spaCy` tiene tres modelos diferentes con la capacidad de clasificar entidades nombradas:

- `es_core_news_sm`
- `es_core_news_md`
- `es_core_news_lg`

La diferencia entre los modelos es su tamaño *small*, *medium* o *large*. Los modelos son redes neuronales convolucionales entrenadas con noticias y documentos de UD Spanish AnCora y WikiNER.

Para el trabajo desarrollado en esta práctica se ha utilizado el modelo de mayor tamaño (`es_core_news_lg`), el cual ofrece mejores eficiencias y, al ser el documento para etiquetar relativamente pequeño, no se demora demasiado en realizar las predicciones.

Además, se ha definido un tokenizador personalizado. La función del tokenizador va a ser importante, ya que nos va a permitir utilizar los mismos tokens que aparecen en el documento *esp.testb* en nuestro entrenamiento. Pudiendo comparar directamente los resultados del modelo con las etiquetas originales. El tokenizador personalizado sustituye al tokenizador por defecto en el pipeline de `spaCy`.

# 4. Resultados

## 4.1 Preparación del tokenizador personalizado y primeros resultados.

Para obtener los primeros resultados se decidió aplicar directamente el modelo. Para ello era necesario separar cada palabra de su etiqueta en el documento a evaluar. Al tener ya el texto tokenizado se decidió utilizar la misma tokenización en el modelo de `spaCy`. Esto simplificaría la evaluación de los resultados.

Como se ha descrito anteriormente, para introducir el texto tokenizado en el modelo se ha utilizado un tokenizador personalizado. Para ello se ha usado el siguiente código:

```python
import spacy
from spacy.tokens import Doc

def custom_tokenizer(patch_file):
    tokens = []
    f = open(patch_file, "r")
    for line in f:
        if line != '\n':  # eliminar las líneas vacías
            tokens.append(line.strip().split(' ')[0])

    f.close()
    return Doc(nlp.vocab, tokens)

nlp.tokenizer = custom_tokenizer
```

Aquí, vemos que en la función `custom_tokenizer` introducimos la dirección del documento que está ya tokenizado. A continuación, leemos el fichero línea a línea evitando las líneas en blanco, ya que no incluyen ninguna palabra. En cada línea separamos la palabra de su etiqueta y la añadimos a la lista tokens. Finalmente, la función devuelve un objeto `Doc` que es el tipo de objeto que espera el siguiente paso del pipeline de `spaCy`. En la última línea le indicamos a `spaCy` que utilice nuestro tokenizador personalizado.

A continuación, simplemente con la siguiente línea de código obtenemos la predicción para los diferentes tokens en un objeto `Doc`:

```python
doc = nlp("esp.testb")
```

Para evaluar los resultados del etiquetado se escribe un documento con el formato que requiere el script de evaluación *conlleval.py* (Para cada línea: token label prediction). Este documento se obtiene con el siguiente código:

```python
# tokens con etiqueta correcta
text_f = open('esp.testb', 'r')
flat = [x.strip().split() for x in list(text_f) if x != '\n']
text_f.close()

# Archivo para ser evaluado por conlleval.py
f = open('solution1', 'w')
for i, item in enumerate(doc):
    line = item.text + ' ' + flat[i][1]
    if item.ent_type_ == '':
        line += ' ' + item.ent_iob_
    else:
        line += ' ' + item.ent_iob_+'-'+item.ent_type_
    f.write(line+'\n')
f.close()
```

Primero preparamos una lista con las etiquetas correctas, extrayéndolas del documento *esp.testb*.

Para escribir el documento iteramos entre todos los tokens de las predicciones. Accedemos a la palabra con el atributo `text`, a la que le añadimos la etiqueta correcta extraída anteriormente. Por último, le añadimos la predicción y su tipo usando los atributos `ent_iob`y `ent_type`. Hay que tener en cuenta que las palabras que no son entidades se clasifican cono *O* dentro del sistema IOB y por tanto, no pertenecen a ningún tipo de entidad.

Utilizando la estrategia descrita se obtuvieron los siguientes resultados:

```bash
processed 51533 tokens with 3558 phrases; found: 3843 phrases; correct: 2417.
accuracy:  67.64%; (non-O)
accuracy:  92.48%; precision:  62.89%; recall:  67.93%; FB1:  65.32
              LOC: precision:  63.34%; recall:  78.41%; FB1:  70.07  1342
             MISC: precision:  12.80%; recall:  26.55%; FB1:  17.27  703
              ORG: precision:  79.91%; recall:  61.07%; FB1:  69.23  1070
              PER: precision:  85.44%; recall:  84.63%; FB1:  85.03  728
```

En los resultados se observa que hemos encontrado más entidades de las que realmente hay (3843 frente a 3558), de las cuales 2417 han sido asignadas correctamente. La exactitud considerando solo las entidades no es demasiado alta (67,64 %). Sin embargo, considerando todos los tokens, la exactitud es de 92,48 % con una puntuación F de 65,32.

Si comparamos estos resultados con los de los participantes en la tarea CoNLL-2002, vemos que se encontrarían en la parte baja de la clasificación. Sin embargo, sí que superarían al modelo base.

Los resultados obtenidos no son malos, pero se necesitaría ajustar el modelo específicamente para este tipo de documentos. Además, es muy posible que la forma en la que se ha tokenizado el documento haya afectado a la predicción, ya que no fue el mismo tipo de tokenizado que fue usado durante el entrenamiento del modelo.

Analizando las predicciones, llamó la atención algunos de los errores que se ven a continuación:

```bash
. O O
En O B-MISC
declaraciones O I-MISC
a O I-MISC
Efe B-ORG I-MISC

. O O
El O B-LOC
Ayuntamiento B-ORG I-LOC
de I-ORG I-LOC
Arévalo I-ORG I-LOC

. O O
Según O B-MISC
el O I-MISC
regidor O I-MISC
, O O
```

Vemos que después de un "." el modelo ha tendido a clasificar erróneamente las siguientes palabras. Parece que el modelo confunde la primera letra en mayúscula con un nombre propio y no con el inicio de una frase.

Esto puede ser debido a que el tokenizador personalizado que hemos usado, elimina la información de los saltos de línea. 

Como primera mejora se planteó el mantener la información de los saltos de línea tras el tokenizado.

### Documentos relativos a este apartado:

- El código utilizado en esta parte se encuentra en el documento *NERdetector_custom_toker_1.py*
- El documento con los resultados de la predicción *solution1*
- Para obtener una lista de todos los errores se ha usado el archivo *detectar_errores.py*
- El documento con la lista de errores *errores1*

## 4.2 Tokenizador con información de los saltos de línea

El tokenizador usado en el apartado anterior descartaba los espacios en blanco entre frases que se habían incluido en el documento *esp.testb.*

Para conservar esta información se ha modificado la función `custom_tokenizer` para que cada vez que encontrase una línea vacía incluyese un token con la *palabra* `\n`. En principio esto debería indicar al modelo cierta delimitación entre las frases.

```python
def custom_tokenizer(patch_file):
    tokens = []
    f = open(patch_file, "r")
    for line in f:
        if line != '\n':
            tokens.append(line.strip().split(' ')[0])
        else:
            tokens.append('\n')  # Mantener los saltos de línea
    f.close()
    return Doc(nlp.vocab, tokens)
```

Si observamos los mismos ejemplos que veíamos en el apartado anterior, ahora el modelo no clasifica erróneamente las palabras justo después del punto:

```bash
. O O
En O O
declaraciones O O
a O O
Efe B-ORG B-ORG

. O O
El O O
Ayuntamiento B-ORG B-LOC
de I-ORG I-LOC
Arévalo I-ORG I-LOC

. O O
Según O O
el O O
regidor O O
, O O
```

Sin embargo, la solución no es 100% eficaz, y hay frases en las que el modelo continúa clasificando erróneamente como entidad la primera palabra. El porqué ocurre esto no está del todo claro, pero se puede concluir que al añadir saltos de líneas entre frases ayuda al modelo a clasificar mejor a las primeras palabras de las frases.

En la siguiente tabla podemos ver los resultados estadísticos tras la mejora:

```bash
processed 51533 tokens with 3558 phrases; found: 3598 phrases; correct: 2489.
accuracy:  69.05%; (non-O)
accuracy:  94.72%; precision:  69.18%; recall:  69.96%; FB1:  69.56
              LOC: precision:  65.02%; recall:  80.07%; FB1:  71.77  1335
             MISC: precision:  22.33%; recall:  27.14%; FB1:  24.50  412
              ORG: precision:  80.64%; recall:  63.36%; FB1:  70.96  1100
              PER: precision:  85.49%; recall:  87.35%; FB1:  86.41  751
```

Vemos que ahora encontramos 3598 entidades frente a las 3843 del apartado anterior. Por tanto, estamos teniendo muchos menos falsos positivos acercándonos al número de positivos reales (3558). Además,  el número de entidades asignadas correctamente aumenta a 2489.

En general, todos los parámetros aumentan, se incrementa la medida F en 4 puntos llegando a 69,56. Este valor sigue siendo bajo comparado con los resultados de la tarea CoNLL-2002, encontrándose entre los 3 últimos de la tabla. 

### Documentos relativos a este apartado:

- El código utilizado en esta parte se encuentra en el documento *NERdetector_custom_toker_2.py*
- El documento con los resultados de la predicción *solution2*
- Para obtener una lista de todos los errores se ha usado el archivo *detectar_errores.py*
- El documento con la lista de errores *errores2*

## 4.3 Transformación de palabras en mayúsculas

Observando los resultados del etiquetado del apartado anterior, se encontró que hay frases en el documento en el que todas las palabras están en mayúsculas y el modelo encuentra dificultades para clasificarlas correctamente. Podemos observar algunos ejemplos de este caso:

```python
LIBANO B-LOC B-MISC
- O O
ISRAEL B-LOC B-MISC
AUMENTAN O O
CHOQUES O O
GUERRILLEROS O B-MISC
LIBANESES O I-MISC
Y O I-MISC
SOLDADOS O O
ISRAELIES O O
Jerusalén B-LOC O
```

Aquí vemos que el modelo ha identificado como entidades a *LÍBANO* e *ISRAEL, pero* ha fallado en clasificarlos correctamente como localizaciones. Por otro lado, ha considerado de forma equívoca *GUERRILLEROS LIBANESES Y* como una entidad. Finalmente, ha fallado al no clasificar a *Jerusalén* como entidad estando esta correctamente escrita.

Vemos que estas frases son posiblemente los títulos de las noticias y el modelo presenta dificultades al clasificar las palabras correctamente. Además no hay punto al final del titular y el modelo falla cuando cambia del titular al texto, como vemos en el caso de *Jerusalén.*

 Para mejorar el etiquetado se podría cambiar todas estas palabras por su equivalente en minúscula. Pero esto podría suponer errores en entidades como nombres y países, que tienen la primera letra en mayúscula, o siglas de entidades, que son completamente en mayúsculas.

Para solucionar esto se podría construir un diccionario con siglas de organizaciones conocidas, o nombres de países y personas y así se podría aliviar este problema.

En este caso se ha optado por analizar en el documento a clasificar las palabras en mayúsculas, y deducir según a la etiqueta que pertenecen si deberían estar en mayúsculas o no.

Se ha utilizado el siguiente código para obtener las diferentes listas de palabras:

```python
org_misc = set()
per = set()
loc = set()
f = open('esp.testb', "r")
for line in f:
    if line != '\n':
        temp = line.strip().split(' ')
        capital_l = temp[0].upper() == temp[0]  # capital words
        # ORG, MISC y LOC en mayusculas como EFE, o ITA
        if temp[1].split('-')[-1] == 'ORG' and capital_l and not temp[0].isnumeric():
            org_misc.add(temp[0])
        if temp[1].split('-')[-1] == 'MISC' and capital_l and not temp[0].isnumeric():
            org_misc.add(temp[0])
        if temp[1].split('-')[-1] == 'LOC' and capital_l and (len(temp[0]) <= 3) and not temp[0].isnumeric():  # ej: ITA
            org_misc.add(temp[0])
        # PER y LOC en mayúsculas que deberían ser primera en mayúsculas y el resto minúscula
        if temp[1].split('-')[-1] == 'PER' and capital_l and not temp[0].isnumeric():
            per.add(temp[0])
        if temp[1].split('-')[-1] == 'LOC' and capital_l and (len(temp[0]) > 3) and not temp[0].isnumeric():
            loc.add(temp[0])
f.close()
```

Primero se seleccionan las palabras que están en mayúsculas y se comprueban si son una *ORG* o *MISC*. Si es así posiblemente sean siglas, como por ejemplo *EFE*, y se añaden al set `org_misc`. A este set se le añade también las *LOC* de pequeño tamaño, que suelen ser siglas de países como *ITA*, **Italia. Por tanto, el set `org_misc` está formado por excepciones, palabras que deben permanecer en mayúsculas.

Por otro lado, tenemos los sets `per` y `loc`. A estos sets les añadimos las palabras etiquetadas como *PER* o *LOC* que están completamente en mayúsculas. Posiblemente estas palabras deberían estar escritas con la primera en mayúsculas y el resto en minúsculas. Esta modificación se realizará en el tokenizador.

El análisis realizado aquí sin duda, está memorizando información del documento a predecir y funcionará bien para este texto y algunos parecidos. Sin embargo, la estrategia se podría generalizar usando listas de iniciales de organizaciones, o listas de nombres de países y personas. Cuanto mayores sean estas listas mejor servirán para otros textos. Además, si se utilizan estructuras tipo set no debería ralentizar el proceso. Aunque, posiblemente lo más interesante sea introducir en el aprendizaje del modelo la capacidad de etiquetar correctamente las palabras en mayúsculas.

 Ya que el objetivo del trabajo no es entrenar el modelo, y preparar listas generales de nombres de países, personas y organizaciones es una tarea ardua, se decidió por su valor educativo (ilustrar si las palabras en mayúsculas afectan al etiquetado) proceder de la manera descrita anteriormente.

Tras la preparación de los diferentes sets, se modificó el tokenizador personalizado para que incluyese la información extraída. A continuación podemos ver como queda el código:

```python
def custom_tokenizer(patch_file):
    tokens = []
    f = open(patch_file, "r")
    for line in f:
        if line != '\n':
            temp = line.strip().split(' ')[0]
            # PER y LOC que debería ser la primera en mayúsculas y el resto minúscula
            if (temp.upper() == temp) and (temp in per):  # PER
                temp = temp[0] + temp[1:].lower()
            elif (temp.upper() == temp) and (temp in loc) and (temp != 'EEUU'):  # LOC
                temp = temp[0] + temp[1:].lower()
            # Resto de palabras que no son ORG o MISC pasar a minúscula
            elif (temp.upper() == temp) and (temp not in org_misc):
                temp = temp.lower()
            tokens.append(temp)
        else:
            tokens.append('\n')  # Mantener los saltos de línea
    f.close()
    return Doc(nlp.vocab, tokens)
```

Vemos que el tokenizador ahora comprueba para cada palabra si está escrita en mayúsculas.

Si es así y pertenece a los sets `per` o `loc`, crea el token con la inicial en mayúsculas y el resto en minúsculas.

Si pertenece al set `org_misc` la palabra está correctamente escrita en mayúsculas y crea el token sin modificarla.

Por último, el resto de palabras en mayúsculas las cambia a minúscula antes de añadirlas a la lista de tokens.

Es importante señalar que el tokenizador no incluye en ningún momento la etiqueta a la que pertenece el token. Será el modelo el que realice el etiquetado, y podremos evaluar si los fallos en el etiquetado anterior eran debidos a que las palabras estaban en mayúsculas.

El resto del código necesario para etiquetar y evaluar los resultados es similar al usado anterior mente.

Podemos ver como clasifica ahora el modelo el ejemplo que pusimos al principio de la sección:

```python
Libano B-LOC B-LOC
- O O
Israel B-LOC B-LOC
aumentan O O
choques O O
guerrilleros O O
libaneses O O
y O O
soldados O O
israelies O O
Jerusalén B-LOC B-LOC
```

Vemos que ahora el modelo es capaz de clasificar correctamente las tres localizaciones, *Líbano*, *Israel* y *Jerusalén.* Además, no clasifica erróneamente las palabras *guerrilleros libaneses y.* Podemos concluir que sin duda el hecho de que las palabras estuviesen en mayúsculas estaba confundiendo al modelo.

A continuación, podemos ver los resultados obtenidos tras las modificaciones realizadas:

```python
processed 51533 tokens with 3558 phrases; found: 3578 phrases; correct: 2526.
accuracy:  69.84%; (non-O)
accuracy:  95.05%; precision:  70.60%; recall:  70.99%; FB1:  70.80
              LOC: precision:  66.30%; recall:  83.30%; FB1:  73.83  1362
             MISC: precision:  24.66%; recall:  26.84%; FB1:  25.71  369
              ORG: precision:  81.04%; recall:  63.50%; FB1:  71.21  1097
              PER: precision:  85.73%; recall:  87.48%; FB1:  86.60  750
```

Si comparamos con los resultados del apartado anterior, vemos que se han producido una cierta mejora en los resultados. El número de entidades encontradas es menor, y se acerca al número de entidades reales, y el número de entidades clasificadas correctamente ha aumentado, llegando a 2526. El resto de parámetros también aumenta ligeramente mejorándose tanto la precisión como el recall, alcanzando una puntuación F de 70,80. Estos valores siguen siendo bajos comparados con los resultados de la tarea CoNLL-2002. Se han conseguido ciertas mejoras al optimizar el etiquetado, pero se siguen cometiendo un cierto número de fallos.

### Documentos relativos a este apartado:

- El código utilizado en esta parte se encuentra en el documento *NERdetector_custom_toker_3.py*
- El documento con los resultados de la predicción *solution3*
- Para obtener una lista de todos los errores se ha usado el archivo *detectar_errores.py*
- El documento con la lista de errores *errores3*

# 5. Conclusiones

En este trabajo se ha usado un modelo del módulo `spaCy` para etiquetar el documento *esp.testb*  de la tarea CoNLL-2002. Además, se ha utilizado un tokenizador personalizado para usar los mismos tokens en nuestro etiquetado que los presentes en el documento *esp.testb.* Los resultados estadísticos del etiquetado se obtuvieron usando el script *conlleval.py.*

Los resultados del etiquetado fueron analizados en busca de posibles mejoras. 

Por un lado, se estudió que la inclusión de tokens con información sobre los saltos de línea entre frases mejoraba el etiquetado. Posiblemente, esto ayuda a evitar confundir las palabras de inicio de frase, que tienen la inicial en mayúsculas, con entidades.

Por otro lado, se ha estudiado como influyen las palabras escritas totalmente en mayúsculas en el etiquetado. El modelo tiende a clasificar erróneamente estas etiquetas, posiblemente porque la distinción entre mayúsculas y minúsculas es importante para encontrar algunas entidades, por ejemplo los nombres de personas o países tienen la inicial en mayúsculas y el resto en minúsculas. Tras realizar una serie de modificaciones en el tokenizado, se comprobó que usando una correcta distribución de mayúsculas y minúsculas el modelo es capaz de etiquetar mejor el documento.

Los resultados estadísticos obtenidos son relativamente bajos comparados con los de la tarea CoNLL-2002, encontrándose en la parte baja de la tabla. Se ha conseguido ciertas mejoras en el etiquetado, pero, aun así los valores se encuentra alejados de los récords en la tarea original. En este trabajo se ha llegado a una puntuación F de 70,80.