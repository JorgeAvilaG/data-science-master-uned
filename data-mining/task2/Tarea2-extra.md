# Tarea 2. Minería de textos: Clustering Parte opcional

# 1. Introducción

En esta parte se va a realizar el clustering de un conjunto de documentos que se encuentran en formato de texto, la colección proporcionada por el equipo docente llamada: *CorpusClustering-Tarea2-parteAdicional.rar*

Por tanto, el primer paso será obtener la representación vectorial de los documentos para que puedan ser utilizados por el software CLUTO, con el que se realizará el clustering.

# 2. Preprocesado de los documentos

El primer paso ha sido identificar los documentos que pertenecen al corpus con el que vamos a trabajar. Son un total de 12 documentos con noticias. Se pueden identificar cuatro temas generales entre los documentos: inundaciones en china, Acontecimientos deportivos, finanzas y Republica Checa en la OTAN.

Para obtener los datos en forma matricial, primero se han eliminado todas las etiquetas XML y nos hemos quedado solo con el campo *TEXT* de los documentos. Como son solo 12 documentos y no son extensos, la eliminación de las etiquetas se ha hecho a mano usando VSCode como procesador de texto.

Como segundo paso, se ha usado el siguiente código en python para agrupar todos los documentos en un solo archivo en el que cada documento sea una fila del archivo.

```python
import os
files = os.listdir()
files = [file for file in files if file[0] in set(['A', 'X'])]

corpus = open('corpus.txt','w')
for file in files:
    f = open(file)
    for line in f:
        corpus.write(line.strip()+' ')
    f.close()
    corpus.write('\n')
corpus.close()
```

# 3. Representación vectorial

Para obtener la representación vectorial de los documentos se ha usado el script en perl doc2mat proporcionado en la web de CLUTO. El script toma como entrada los documentos con el formato preparado en el apartado anterior, con un documento en cada fila, y devuelve la representación vectorial de los documentos.

Para realizar la representación vectorial el script utiliza una función de pesado local que tiene en cuenta solo la frecuencia de cada término en cada documento por separado, sin tener en cuenta la frecuencia global del término. Es una función de peso bastante simple que no tiene en cuenta algunos aspectos como la longitud del documento o la distribución de términos en los otros documentos del corpus. Sin embargo, usando esta función se han obtenido resultados de clustering muy buenos, por tanto no es necesario el uso de funciones más complejas para la tarea que nos atañe. Matemáticamente podemos describir la función de pesado como:

$$TF(\vec t_i,\vec d_j)=f_{ij}$$

El número de veces que el término $t_i$ ocurre en el documento $d_j$.

Para ejecutar el script se usa el siguiente comando:

```python
perl .\doc2mat .\corpus.txt .\corpus.mat
```

Generando el archivo *corpus.mat* con la representación vectorial de los documentos lista para ser utilizada en CLUTO.

También se genera automáticamente el archivo *corpus.mat.clabel* que contiene una lista con todos los términos codificados en la representación vectorial.

Por último, manualmente se ha preparado el archivo *corpus.mat.rclass* que contiene, en orden, la categoría que se le ha asignado a cada documento. Se han usado cuatro categorías: *floods, CzechArmy, Bowls* y *Finantial.* El contenido del archivo es el siguiente:

```python
floods
floods
floods
CzechArmy
Bowls
Bowls
Finantial
Finantial
Bowls
CzechArmy
CzechArmy
CzechArmy
```

# 4. Clustering con CLUTO

Para la ejecución en CLUTO se ha usado los valores por defecto del algoritmo de clustering, y de las funciones de similitud y criterio. Sin embargo, se han añadido dos opciones. `showfeatures` que ofrece un resumen de los términos más influyentes en el clustering, y también se han usado la opción `rclassfile='corpus.mat.rclass'` para obtener medidas externas de calidad usando la clasificación que hemos preparado en el apartado anterior. Por último, hemos indicado 4 como el número de clústeres a identificar que se corresponderían con las 4 categorías en los que fueron asignados los documentos.

Para ejecutar el programa se ha usado el siguiente comando:

```python
.\vcluster.exe -showfeatures -rclassfile='corpus.mat.rclass' .\corpus.mat 4
```

La salida del programa es la siguiente:

```python
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\corpus.mat, #Rows: 12, #Columns: 1121, #NonZeros: 1680

Options ----------------------------------------------------------------------
  CLMethod=RB, CRfun=I2, SimFun=Cosine, #Clusters: 4
  RowModel=None, ColModel=IDF, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=I2, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

------------------------------------------------------------------------
4-way clustering: [I2=9.00e+000] [12 of 12], Entropy: 0.000, Purity: 1.000
------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | floo Czec Bowl Fina
------------------------------------------------------------------------
  0     2 +0.747 +0.000 +0.017 +0.006 0.000 1.000 |    0    0    0    2
  1     3 +0.628 +0.015 +0.010 +0.001 0.000 1.000 |    0    0    3    0
  2     3 +0.599 +0.008 +0.017 +0.002 0.000 1.000 |    3    0    0    0
  3     4 +0.413 +0.022 +0.012 +0.002 0.000 1.000 |    0    4    0    0
------------------------------------------------------------------------
--------------------------------------------------------------------------------
4-way clustering solution - Descriptive & Discriminating Features...
--------------------------------------------------------------------------------
Cluster   0, Size:     2, ISim: 0.747, ESim: 0.017
      Descriptive:  ebai 26.6%, auction 11.2%, fraud  5.4%, consum  3.5%, compani  2.7%
   Discriminating:  ebai 14.0%, auction  5.9%, bowl  4.9%, flood  3.7%, czech  3.2%

Cluster   1, Size:     3, ISim: 0.628, ESim: 0.010
      Descriptive:  bowl 30.2%, game 10.0%, bc  5.6%, fiesta  4.3%, rate  4.3%
   Discriminating:  bowl 15.6%, game  5.2%, flood  4.2%, czech  3.7%, bc  2.9%

Cluster   2, Size:     3, ISim: 0.599, ESim: 0.017
      Descriptive:  flood 24.1%, yangtz  9.5%, river  9.2%, china  2.9%, water  2.6%
   Discriminating:  flood 12.7%, bowl  5.7%, yangtz  5.0%, river  4.8%, czech  3.8%

Cluster   3, Size:     4, ISim: 0.413, ESim: 0.012
      Descriptive:  czech 17.1%, republ  6.4%, nato  5.3%, countri  3.7%, hungari  3.0%
   Discriminating:  czech  8.9%, bowl  6.0%, flood  4.6%, republ  3.4%, ebai  2.8%
--------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.002 sec
   Clustering:                            0.001 sec
   Reporting:                             0.025 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                  131072 bytes
   Current memory used:                   61176 bytes
********************************************************************************
```

Analizando la salida del programa, vemos que se ha obtenido un valor de 9 para la función de criterio, se han clasificado todos los documentos y la ejecución del programa ha sido rápida. Sobre los parámetros de calidad externos generales, vemos que el clustering ha sido perfecto, con una entropía de 0 y  una pureza de 1. Por tanto, todos los documentos se han asignado a la categoría esperada. 

En la descripción de los diferentes clústeres, vemos que todos tienen una pureza de 1 y una entropía de cero. En la matriz de la derecha vemos que cada categoría ha sido asignada a un solo clúster y podemos ver cuantos individuos hay en cada clúster. Por ejemplo, el clúster 0 contiene los dos documentos con temática de finanzas. Observamos que las medidas de similitud son altas en todos los clústeres, y las medidas de similitud externa muy bajas. 

En la salida correspondiente a la opción `showfeatures`, vemos que para cada clúster se ofrece una lista de los términos que más influyen a la hora de caracterizar o describir el clúster, y una segunda lista de los términos más importante que sirven para diferenciar el clúster del resto. Vemos que sobre todo los términos descriptivos son muy característicos para cada clase, por ejemplo en el clúster 2, correspondiente a las noticias de inundaciones en China, estos términos son flood, Yangtz, river, China y water.

# 5. Conclusiones

En este trabajo se ha realizado el proceso de clustering de una colección de documentos pasando por todos los pasos. Se ha empezado realizando una limpieza y formateo de los documentos para adaptarlos al formato requerido para los scripts a utilizar. Se ha obtenido la representación vectorial de los documentos, y finalmente se ha realizado el clustering.

Los resultados obtenidos han sido muy buenos, obteniéndose una clasificación perfecta de los documentos en cada clúster correspondiéndose a las clases designadas manualmente.