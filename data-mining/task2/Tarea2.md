# Tarea 2. Minería de textos: Clustering

# 1. Introducción

En esta tarea se va a estudiar el clustering de documentos, utilizando el software CLUTO. Esta es una herramienta que permite el clustering de colecciones usando diferentes tipos de algoritmos, además del análisis y caracterización de los clústeres encontrados.  Dentro de los diferentes tipos de algoritmos que se pueden usar, aquí vamos a tratar algoritmos de partición y aglomerativos. 

[CLUTO - Software for Clustering High-Dimensional Datasets](http://glaros.dtc.umn.edu/gkhome/cluto/cluto/overview)

## 1.1 Colección de datos

En esta práctica se va a estudiar la colección de datos *re0* que se puede obtener desde la propia web de CLUTO: 

[CLUTO download | Karypis Lab](http://glaros.dtc.umn.edu/gkhome/cluto/cluto/download)

La colección *re0* forma parte de un conjunto de 15 datasets diferentes. En particular la colección *re0,* un subconjunto de la colección Reuters-21578, está formado por 1504 documentos, con 2886 términos y 13 clases. Cada documento en la colección pertenece únicamente a una clase.

Los documentos que forman parte de esta colección son un conjunto de noticias publicadas por Reuters en 1987. Estos documentos fueron categorizados y agrupados por personal de Reuteurs y Carnegie Group en 1987. Más tarde, en 1990, estos documentos se hicieron públicos y disponibles para propósitos de investigación. La versión *Reuters-21578, Distribution 1.0*  fue desarrollada en 1996, por Steve Finch y David D. Lewis tras una limpieza de los datos. Cada documento puede pertenecer a una o más categorías, que se encuentran agrupadas en los tipos: EXCHANGES, ORGS, PEOPLE, PLACES o TOPICS. Cada tipo contiene un número de diferentes categorías. La tarea de categorización de textos consiste en predecir las posibles categorías a las que pertenece un documento. En la colección *re0,* se han seleccionado documentos que pertenecen solo a una categoría, por tanto se podrían agrupar en diferentes clústeres, con un clúster para cada categoría.

Las clases contenidas en la colección *re0* son: *housing, money, trade, reserves, cpi, interest, gnp, retail, ipi, jobs, lei, bop* y *wpi.*

# 2. Representación de los documentos

Cada documento en la colección con la que vamos a trabajar está representado como un vector en el espacio de los términos  (Modelo del espacio vectorial). En el que cada valor en el vector es la frecuencia ponderada de cada término o rasgo. Se realiza el ponderado para disminuir la influencia de términos que aparezca frecuentemente en muchos documentos, estos términos que aparecen a menudo tienen poco poder discriminante y es preferible disminuir su influencia en el modelo. Además, la longitud de cada vector se normaliza a 1 para tener en cuenta que los documentos pueden tener diferente longitud. En resumen cada documento es un vector de longitud 1 en el espacio de los términos, formado por la frecuencia ponderada de cada término.

El cálculo de la frecuencia ponderada es un tipo de función de pesado global, ya que tiene en cuenta información de todos los documentos de la colección para obtener el peso de cada rasgo en los documentos. En particular, esta función se conoce como *Frecuencia del término x Frecuencia inversa de Documento* (TF-IDF). Con esta función se alcanzan valores altos cuando la frecuencia del término es alta en el documento dado y es pequeña en general en la colección completa. Sin embargo, cuando un término es popular y aparece en muchos documentos su valor tenderá a cero.

También podemos comentar que en la colección se han realizado una serie de tareas de reducción de rasgos. Se ha usado una lista de palabras vacías (stop-list) para eliminar palabras comunes, el algoritmo de Porter para suffix-stripping, y se eliminó cualquier término que apareciese en menos de dos documentos.

# 3. Parámetros utilizados

Para la realización de la práctica se han elegido dos algoritmos de clustering. El algoritmo de partición elegido es `direct`. Este algoritmo es capaz de encontrar simultáneamente todos los clústeres. Aunque computacionalmente es más lento que otros métodos, para valores pequeños de k (entre 10 y 20) en el manual indica que da mejores resultados que los otros algoritmos de partición. Como para este problema tenemos 13 clústeres he decidido trabajar con este algoritmo.

El algoritmo de aglomeración utilizado es el `agglo`. Este método usa el paradigma de aglomeración para ir agrupando las instancias según la función criterio, hasta obtener el número de clústeres deseados.

También se han seleccionado dos funciones de similitud, `cos` y `corr`. De las 4 funciones posibles se han elegido estas dos porque son las únicas que podemos aplicar a los algoritmos utilizados. La función `cos` evalúa la similitud usando la función coseno. Por otro lado, la función `corr` mide la similitud usando el coeficiente de correlación.

Por último, las dos funciones de criterio seleccionadas han sido `i2` y `slink`. La función de optimización i2 es la estándar en el programa y consiste en la maximización de la siguiente fórmula para encontrar los clústeres:

$$maximize \sum^k_{i=1}\sqrt{\sum_{v,u \in S_i}sim(v,u)}$$

Por otro lado, la función `slink` usa el criterio del enlace único. Esta función solo se podrá utilizar con el algoritmo `agglo`. Se ha decidido utilizar esta función porque usada junto al algoritmo `agglo` es adecuada para encontrar clústeres de tipo *transitivo*, mientras que el resto de combinaciones serán más adecuadas para encontrar clústeres *globulares.* De este modo podemos estudiar algoritmos muy diferentes que nos van a permitir adaptarnos al tipo de clústeres que podamos encontrar en los datos. 

En resumen se han realizado las siguientes seis ejecuciones combinando las opciones descritas:

```r
.\vcluster.exe -clmethod=direct -sim=cos -crfun=i2 -rclassfile='re0.mat.rclass' .\re0.mat 13
.\vcluster.exe -clmethod=direct -sim=corr -crfun=i2 -rclassfile='re0.mat.rclass' .\re0.mat 13
.\vcluster.exe -clmethod=agglo -sim=cos -crfun=i2 -rclassfile='re0.mat.rclass' .\re0.mat 13
.\vcluster.exe -clmethod=agglo -sim=cos -crfun=slink -rclassfile='re0.mat.rclass' .\re0.mat 13
.\vcluster.exe -clmethod=agglo -sim=corr -crfun=i2 -rclassfile='re0.mat.rclass' .\re0.mat 13
.\vcluster.exe -clmethod=agglo -sim=corr -crfun=slink -rclassfile='re0.mat.rclass' .\re0.mat 13
```

Además se ha usado el parámetro `classfile` para indicar el archivo en el que se encuentran las clases a las que pertenece cada documento. Esto permite al algoritmo comparar los clústeres encontrados con las clases reales en las que se divide los datos, obteniéndose las medidas externas además de las internas como parámetros para el estudio de la calidad de los modelos.

# 4. Resultados

A continuación se comentan los resultados de las diferentes ejecuciones.

- Para el algoritmo `direct` con la función de similitud `cos` y la función de criterio `i2`:

```r
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\re0.mat, #Rows: 1504, #Columns: 2886, #NonZeros: 77808

Options ----------------------------------------------------------------------
  CLMethod=Direct, CRfun=I2, SimFun=Cosine, #Clusters: 13
  RowModel=None, ColModel=IDF, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=I2, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------
13-way clustering: [I2=6.03e+002] [1504 of 1504], Entropy: 0.388, Purity: 0.648
---------------------------------------------------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | hous mone trad rese  cpi inte  gnp reta  ipi jobs  lei  bop  wpi
---------------------------------------------------------------------------------------------------------------------
  0   112 +0.485 +0.117 +0.034 +0.008 0.127 0.929 |    0  104    4    0    0    3    0    0    0    0    0    1    0
  1    99 +0.325 +0.100 +0.033 +0.011 0.249 0.758 |    0   75    1    0    1   22    0    0    0    0    0    0    0
  2    49 +0.200 +0.073 +0.027 +0.008 0.822 0.306 |    2   15    5    2    4    0    0    3    2    6    2    7    1
  3   110 +0.178 +0.056 +0.034 +0.014 0.040 0.982 |    0  108    1    0    0    1    0    0    0    0    0    0    0
  4    67 +0.158 +0.042 +0.027 +0.010 0.403 0.507 |    0   34   25    0    2    6    0    0    0    0    0    0    0
  5   174 +0.156 +0.044 +0.033 +0.010 0.829 0.270 |   11   25    2    2   47    3    3   14   23   22    8    0   14
  6   160 +0.160 +0.048 +0.044 +0.011 0.579 0.331 |    1   53   46   28    0    4    1    0    0    0    0   27    0
  7    87 +0.144 +0.043 +0.031 +0.010 0.097 0.943 |    0    4   82    0    0    0    1    0    0    0    0    0    0
  8   122 +0.125 +0.039 +0.029 +0.014 0.129 0.910 |    0   10    1    0    0  111    0    0    0    0    0    0    0
  9   113 +0.125 +0.031 +0.039 +0.014 0.346 0.566 |    0   64    0    6    0   42    1    0    0    0    0    0    0
 10   136 +0.091 +0.024 +0.027 +0.009 0.222 0.868 |    1   11  118    2    0    1    1    0    0    1    0    1    0
 11   141 +0.101 +0.028 +0.039 +0.013 0.676 0.496 |    1   12   16    1    6    7   70    3   12   10    1    2    0
 12   134 +0.086 +0.030 +0.035 +0.013 0.359 0.694 |    0   93   18    1    0   19    3    0    0    0    0    0    0
---------------------------------------------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.055 sec
   Clustering:                            0.172 sec
   Reporting:                             0.010 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                 1835008 bytes
   Current memory used:                  947064 bytes
********************************************************************************
```

Dentro de las medidas internas de calidad tenemos que la función de criterio alcanza un valor de 6.03e+002, como es una función de maximización, cuando mayor sea este valor mejores son los resultados. También vemos que se han asignado todos los documentos a clústeres (1504 of 1504). Como parámetros extra de calidad interna, tenemos la columna ISim y su desviación estándar ISdev, que nos indica la similitud interna media dentro de cada clúster. Un valor alto para este parámetro nos indica que tenemos un cluster con individuos muy parecidos entre ellos. Por otro lado, tenemos la columna ESim y su desviación estándar ESdev. Estas columnas nos indican la similitud externa media, es decir, como de similares son los miembros de un clúster con el resto de entidades. Valores bajos para este parámetro nos indica que el clúster está bien diferenciado del resto.

Como medidas externas de calidad tenemos la entropía y la pureza. La entropía hace referencia a la cantidad de documentos que pertenecen a clases diferentes, pero están en el mismo clúster, y la pureza aumenta con clústeres en el que todos sus miembros pertenecen a la misma clase. Podemos obtener estas medidas de manera global y específicas para cada clúster. Sobre los resultados globales tenemos una pureza modesta de 0.648 y una entropía de 0.388. Entre los diferentes clúster podemos destacar algunos como 0, 3, 7 y 8 que presentan unas purezas altas (>0.9) y entropías bajas. Podemos comprobar en la tabla de la derecha que efectivamente estos clústeres tienen principalmente individuos de una sola clase. Sin embargo, los clústeres no han conseguido una Isim alta con este modelo y la mayoría se encuentra alrededor de 0.1, siendo bastante baja para clústeres muy buenos como el 7 y 8. Por otro lado, la ESim es en general baja para todos los clústeres con valores muy parecidos (0.03-0.04). 

El algoritmo ha sido rápido con un tiempo de ejecución de 0.172 seg.

- Para el algoritmo `direct` con la función de similitud `corr` y la función de criterio `i2`:

```r
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\re0.mat, #Rows: 1504, #Columns: 2886, #NonZeros: 77808

Options ----------------------------------------------------------------------
  CLMethod=Direct, CRfun=I2, SimFun=CorrCoef, #Clusters: 13
  RowModel=None, ColModel=None, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=I2, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------
13-way clustering: [I2=8.64e+002] [1504 of 1504], Entropy: 0.378, Purity: 0.668
---------------------------------------------------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | hous mone trad rese  cpi inte  gnp reta  ipi jobs  lei  bop  wpi
---------------------------------------------------------------------------------------------------------------------
  0   107 +0.642 +0.102 +0.099 +0.028 0.091 0.953 |    0  102    3    0    0    1    0    0    0    0    0    1    0
  1    61 +0.537 +0.124 +0.084 +0.030 0.323 0.639 |    0   39    1    0    2   19    0    0    0    0    0    0    0
  2   166 +0.397 +0.092 +0.103 +0.026 0.777 0.289 |   11   24    1    0   48    1    2   14   23   24    4    0   14
  3    79 +0.370 +0.102 +0.080 +0.033 0.000 1.000 |    0   79    0    0    0    0    0    0    0    0    0    0    0
  4    89 +0.353 +0.096 +0.091 +0.034 0.364 0.742 |    0   66   10    3    0    0    1    0    0    4    0    5    0
  5   171 +0.364 +0.074 +0.116 +0.032 0.259 0.789 |    1   30    1    0    1  135    1    0    0    2    0    0    0
  6   183 +0.330 +0.083 +0.095 +0.028 0.639 0.311 |    2   45   57   36    0    4    5    0    1    2    0   31    0
  7    88 +0.291 +0.065 +0.070 +0.026 0.121 0.920 |    0    6   81    0    0    0    1    0    0    0    0    0    0
  8   149 +0.317 +0.079 +0.116 +0.032 0.742 0.423 |    2   20    6    0    8   16   63    6   12    7    7    1    1
  9    75 +0.238 +0.073 +0.069 +0.030 0.264 0.800 |    0   60    8    0    0    6    1    0    0    0    0    0    0
 10   148 +0.208 +0.056 +0.054 +0.026 0.073 0.959 |    0    5  142    0    0    1    0    0    0    0    0    0    0
 11   101 +0.240 +0.058 +0.108 +0.033 0.419 0.584 |    0   59    5    2    0   30    4    0    1    0    0    0    0
 12    87 +0.155 +0.042 +0.073 +0.034 0.258 0.839 |    0   73    4    1    1    6    2    0    0    0    0    0    0
---------------------------------------------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.042 sec
   Clustering:                            6.607 sec
   Reporting:                             0.061 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                35979264 bytes
   Current memory used:                  947320 bytes
********************************************************************************
```

La diferencia con el algoritmo anterior es que en este caso hemos usado la función de similitud `corr`.

Sobre las medidas internas de calidad la función de criterio alcanza 8.64e+002, mayor que en el caso anterior. Además se clasifican todos los documentos. En general, los valores de ISim son más altos en este caso, aunque están acompañados de valores de Esim más altos también.

Sobre las medidas externas, tenemos una entropía de 0.378 y una pureza de 0.668, valores prácticamente idénticos comparando con la ejecución anterior. En este caso tenemos los clústeres 0, 3, 7 y 10 con valores de pureza mayores de 0.9. Es interesante que el clúster 3 tenga una pureza de 1. Si miramos la tabla de la derecha vemos que este clúster solo contiene documentos con la etiqueta *money*. Sin embargo, la etiqueta *money* se encuentra muy dispersa entre otros clústeres y por ejemplo, en el clúster 1 la etiqueta mayoritaria también es *money*.  

Sobre la ejecución este modelo ha sido mucho más lento, tardando 6.607 seg. 

- Para el algoritmo `agglo` con la función de similitud `cos` y la función de criterio `i2`:

```r
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\re0.mat, #Rows: 1504, #Columns: 2886, #NonZeros: 77808

Options ----------------------------------------------------------------------
  CLMethod=AGGLO, CRfun=I2, SimFun=Cosine, #Clusters: 13
  RowModel=None, ColModel=IDF, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=I2, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------
13-way clustering: [I2=5.79e+002] [1504 of 1504], Entropy: 0.443, Purity: 0.577
---------------------------------------------------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | hous mone trad rese  cpi inte  gnp reta  ipi jobs  lei  bop  wpi
---------------------------------------------------------------------------------------------------------------------
  0   104 +0.512 +0.115 +0.034 +0.005 0.037 0.981 |    0  102    0    0    0    2    0    0    0    0    0    0    0
  1   129 +0.136 +0.052 +0.045 +0.015 0.576 0.372 |    1   40   48   19    0    2    1    0    2    0    0   16    0
  2    78 +0.126 +0.042 +0.027 +0.009 0.497 0.500 |    0   39   23    0    3    8    1    0    0    4    0    0    0
  3   250 +0.086 +0.028 +0.038 +0.014 0.870 0.224 |   14   48   18    5    9   15   56   13   24   32    7    9    0
  4    57 +0.548 +0.095 +0.036 +0.007 0.248 0.667 |    0   38    0    0    0   19    0    0    0    0    0    0    0
  5    96 +0.199 +0.045 +0.034 +0.015 0.091 0.938 |    0   90    0    0    0    6    0    0    0    0    0    0    0
  6    82 +0.194 +0.058 +0.038 +0.014 0.567 0.646 |    1   53    4    3    3    2    0    3    2    2    2    6    1
  7   176 +0.091 +0.029 +0.027 +0.010 0.221 0.858 |    0   17  151    0    1    0    2    0    3    1    1    0    0
  8    74 +0.168 +0.036 +0.043 +0.013 0.568 0.500 |    0   37    7   11    0   10    3    0    0    0    0    6    0
  9   177 +0.066 +0.020 +0.037 +0.014 0.518 0.424 |    0   75   64    3    0   20   11    1    2    0    1    0    0
 10    67 +0.258 +0.065 +0.038 +0.008 0.404 0.657 |    0    1    1    0   44    0    0    3    4    0    0    0   14
 11    49 +0.245 +0.068 +0.027 +0.012 0.039 0.980 |    0    1    0    0    0   48    0    0    0    0    0    0    0
 12   165 +0.081 +0.026 +0.037 +0.014 0.374 0.527 |    0   67    3    1    0   87    6    0    0    0    0    1    0
---------------------------------------------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.048 sec
   Clustering:                            0.253 sec
   Reporting:                             0.025 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                23920640 bytes
   Current memory used:                  959288 bytes
********************************************************************************
```

Pasamos a estudiar el algoritmo `agglo`. Para esta ejecución obtenemos la mejor hasta ahora de los valores para la función de criterio 5.79e+002. Sobre el resto de valores se ha obtenido un clustering un poco peor que en los casos anteriores con valores de ISim en general más bajos. Sobre la entropía y la pureza tenemos peores valores también con entropías más altas y purezas más bajas. Con respecto a la matriz de la derecha los resultados se ven bastante dispersos, y salvo en pequeñas excepciones como el clúster 0, hay mezclas de varias clases en los clústeres.

- Para el algoritmo `agglo` con la función de similitud `cos` y la función de criterio `slink`:

```r
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\re0.mat, #Rows: 1504, #Columns: 2886, #NonZeros: 77808

Options ----------------------------------------------------------------------
  CLMethod=AGGLO, CRfun=SLINK, SimFun=Cosine, #Clusters: 13
  RowModel=None, ColModel=IDF, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=SLINK, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------
13-way clustering: [SLINK=0.00e+000] [1504 of 1504], Entropy: 0.706, Purity: 0.410
---------------------------------------------------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | hous mone trad rese  cpi inte  gnp reta  ipi jobs  lei  bop  wpi
---------------------------------------------------------------------------------------------------------------------
  0     1 +1.000 +0.000 +0.013 +0.000 0.000 1.000 |    0    0    0    0    0    1    0    0    0    0    0    0    0
  1     1 +1.000 +0.000 +0.029 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  2     4 +0.675 +0.113 +0.025 +0.019 0.405 0.500 |    0    2    1    0    0    0    0    0    1    0    0    0    0
  3     1 +1.000 +0.000 +0.054 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  4     1 +1.000 +0.000 +0.026 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  5     1 +1.000 +0.000 +0.035 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  6     1 +1.000 +0.000 +0.016 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  7     1 +1.000 +0.000 +0.037 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  8     1 +1.000 +0.000 +0.032 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  9     1 +1.000 +0.000 +0.026 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
 10     2 +0.946 +0.000 +0.040 +0.011 0.000 1.000 |    0    0    0    0    0    0    2    0    0    0    0    0    0
 11     1 +1.000 +0.000 +0.041 +0.000 0.000 1.000 |    0    0    0    1    0    0    0    0    0    0    0    0    0
 12  1488 +0.045 +0.016 +0.031 +0.013 0.712 0.405 |   16  602  314   41   60  218   78   20   36   39   11   38   15
---------------------------------------------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.050 sec
   Clustering:                            0.172 sec
   Reporting:                             0.017 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                23920640 bytes
   Current memory used:                  959288 bytes
********************************************************************************
```

Los resultados de este clustering son muy malos. Vemos que tenemos un clúster donde se encuentran la mayoría de los documentos, el clúster 12, con 1488 entidades. El resto de clúster tienen entre 1 y 4 entidades. Por la matriz de las clases vemos que este clustering es totalmente incorrecto y se han obtenido puntuaciones muy malas.

En este intento se ha usado la función de criterio `slink` que sirve especialmente para clústeres de tipo transitivo. Por los resultados obtenidos con esta ejecución podemos concluir que los datos siguen el paradigma de clústeres transitivos. 

- Para el algoritmo `agglo` con la función de similitud `corr` y la función de criterio `i2`:

```r
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\re0.mat, #Rows: 1504, #Columns: 2886, #NonZeros: 77808

Options ----------------------------------------------------------------------
  CLMethod=AGGLO, CRfun=I2, SimFun=CorrCoef, #Clusters: 13
  RowModel=None, ColModel=None, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=I2, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------
13-way clustering: [I2=8.36e+002] [1504 of 1504], Entropy: 0.374, Purity: 0.651
---------------------------------------------------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | hous mone trad rese  cpi inte  gnp reta  ipi jobs  lei  bop  wpi
---------------------------------------------------------------------------------------------------------------------
  0   147 +0.264 +0.077 +0.119 +0.030 0.731 0.463 |    3   16    9    0    8   10   68    6    9   11    4    2    1
  1   131 +0.204 +0.060 +0.056 +0.026 0.125 0.939 |    1    3  123    0    0    0    0    0    2    1    0    1    0
  2    59 +0.567 +0.101 +0.089 +0.025 0.245 0.678 |    0   40    0    0    0   19    0    0    0    0    0    0    0
  3   100 +0.679 +0.076 +0.102 +0.028 0.022 0.990 |    0   99    0    0    0    1    0    0    0    0    0    0    0
  4   119 +0.176 +0.058 +0.100 +0.039 0.340 0.639 |    0   76    3    0    0   35    5    0    0    0    0    0    0
  5   172 +0.345 +0.082 +0.096 +0.028 0.561 0.349 |    1   41   60   33    0    1    1    0    0    0    0   35    0
  6   166 +0.118 +0.038 +0.079 +0.039 0.467 0.542 |    0   90   49    8    4   11    4    0    0    0    0    0    0
  7    60 +0.395 +0.093 +0.084 +0.031 0.128 0.917 |    0   55    0    0    0    1    0    0    0    4    0    0    0
  8   185 +0.375 +0.096 +0.103 +0.029 0.790 0.259 |   11   36    1    0   48    4    1   14   26   23    7    0   14
  9   161 +0.389 +0.069 +0.121 +0.027 0.198 0.814 |    0   29    0    1    0  131    0    0    0    0    0    0    0
 10    71 +0.306 +0.068 +0.067 +0.022 0.079 0.958 |    0    2   68    0    0    0    1    0    0    0    0    0    0
 11    50 +0.526 +0.068 +0.087 +0.033 0.000 1.000 |    0   50    0    0    0    0    0    0    0    0    0    0    0
 12    83 +0.230 +0.064 +0.081 +0.031 0.200 0.855 |    0   71    6    0    0    6    0    0    0    0    0    0    0
---------------------------------------------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.046 sec
   Clustering:                            6.275 sec
   Reporting:                             0.045 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                79626240 bytes
   Current memory used:                  959368 bytes
********************************************************************************
```

En esta ejecución se han usado la función de similitud `corr`, obteniéndose resultados muy parecidos a los obtenidos con el algoritmo `direct`. Resaltar también aquí, los altos tiempos de ejecución (6.275 seg) debidos a la función de similitud `corr`.

- Para el algoritmo `agglo` con la función de similitud `corr` y la función de criterio `slink`:

```r
********************************************************************************
vcluster (CLUTO 2.1.2) Copyright 2001-06, Regents of the University of Minnesota

Matrix Information -----------------------------------------------------------
  Name: .\re0.mat, #Rows: 1504, #Columns: 2886, #NonZeros: 77808

Options ----------------------------------------------------------------------
  CLMethod=AGGLO, CRfun=SLINK, SimFun=CorrCoef, #Clusters: 13
  RowModel=None, ColModel=None, GrModel=SY-DIR, NNbrs=40
  Colprune=1.00, EdgePrune=-1.00, VtxPrune=-1.00, MinComponent=5
  CSType=Best, AggloFrom=0, AggloCRFun=SLINK, NTrials=10, NIter=10

Solution ---------------------------------------------------------------------

---------------------------------------------------------------------------------------------------------------------
13-way clustering: [SLINK=0.00e+000] [1504 of 1504], Entropy: 0.708, Purity: 0.409
---------------------------------------------------------------------------------------------------------------------
cid  Size  ISim  ISdev   ESim  ESdev  Entpy Purty | hous mone trad rese  cpi inte  gnp reta  ipi jobs  lei  bop  wpi
---------------------------------------------------------------------------------------------------------------------
  0     1 +1.000 +0.000 +0.046 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  1     1 +1.000 +0.000 +0.016 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  2     1 +1.000 +0.000 +0.055 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  3     1 +1.000 +0.000 +0.059 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  4     1 +1.000 +0.000 +0.032 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  5     1 +1.000 +0.000 +0.079 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
  6     1 +1.000 +0.000 +0.037 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  7     1 +1.000 +0.000 +0.046 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  8     1 +1.000 +0.000 +0.062 +0.000 0.000 1.000 |    0    0    1    0    0    0    0    0    0    0    0    0    0
  9     1 +1.000 +0.000 +0.050 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
 10     1 +1.000 +0.000 +0.046 +0.000 0.000 1.000 |    0    1    0    0    0    0    0    0    0    0    0    0    0
 11     1 +1.000 +0.000 +0.011 +0.000 0.000 1.000 |    0    0    0    0    0    1    0    0    0    0    0    0    0
 12  1492 +0.114 +0.041 +0.045 +0.021 0.714 0.404 |   16  603  313   42   60  218   80   20   37   39   11   38   15
---------------------------------------------------------------------------------------------------------------------

Timing Information -----------------------------------------------------------
   I/O:                                   0.051 sec
   Clustering:                            6.186 sec
   Reporting:                             0.058 sec
Memory Usage Information -----------------------------------------------------
   Maximum memory used:                79626240 bytes
   Current memory used:                  959272 bytes
********************************************************************************
```

De nuevo en este caso al usar la función de criterio `slink`se obtiene un clustering muy malo, en el que la mayoría de las instancias se encuentran agrupadas en un único clúster. 

![images/ejecuciones.jpg](images/ejecuciones.jpg)

En la tabla resumen podemos observar algunas características generales. Las ejecuciones que han usado la función de similitud `corr` junto a la función de criterio `I2`, son las ejecuciones con mejores valores de Pureza, Entropía y puntuación de la función de criterio. Independientemente del algoritmo utilizado.

Por otro lado, la ejecución con la función de similitud `cos` junto a la función de criterio `I2` para el algoritmo `Direct` obtiene valores muy cercanos de entropía y pureza comparados con los modelos descritos en el párrafo anterior. Mientras que usando el algoritmo `AGGLO` sí que presenta, para estas mismas funciones de similitud y criterio, peores resultados. 

También podemos destacar que la función de criterio `SLINK` no ha sido capaz de realizar un clustering apropiado de los datos. Esto nos da información de que posiblemente los datos utilizados en esta práctica sigan un paradigma de clústeres globulares y no de clústeres transitivos. 

Por último, la función de similitud `corr` ha dado lugar a las ejecuciones más lentas, sobre 6 segundos, comparado con los 0.1-0.2 segundos para las ejecuciones con la función `cos`. 

# 5. Conclusiones

En este trabajo se ha estudiado el clustering de documentos mediante la aplicación CLUTO. Se ha realizado el clustering de una colección de documentos que ya se encontraba en la representación adecuada en forma matricial para ser usada por el programa. Se han realizado 6 ejecuciones diferentes cambiando los algoritmos de clustering, y las funciones de similitud y criterio. Se ha realizado un análisis de los resultados obtenidos, atendiendo tanto a medidas de calidad internas como externas. La ejecución con mejores resultados ha utilizado el algoritmo `direct`, con la función de similitud `corr` y la función de criterio `I2`. También se han estudiado los tiempos de ejecución siendo bastante mayores cuando se usa la función de similitud `corr`.