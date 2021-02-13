# Modelado estadístico de datos Práctica 2

# Jorge Pablo Ávila Gómez

### Ejercicio 1. **(2 puntos)**

**Se ha realizado un estudio para ver si la utilización de un biomarcador influye a la hora de detectar la presencia de nódulos infectados (rta=1: s, rta=0: no). Para ello se han tomado 20 individuos con nódulos infectados y 33 sin nódulos infectados. Los datos experimentales se pueden encontrar en el fichero *le10.txt* alojado en el curso virtual. Se pide dibujar la curva *roc* y calcular el *auc*: ¿Cómo se puede utilizar este biomarcador en la práctica? ¿Se puede utilizar para predecir la presencia de nódulos infectados?**

Existen numerosas librerías en *R p*ara dibujar la curva *roc* y calcular el *auc*, para este ejercicio se han usado las librerías `ROCit` y `PROC`. Con el siguiente código se puede dibujar la curva *roc* usando la librería `ROCit`:

```r
rm(list=ls())
data = read.table('le10.txt', header = T)
attach(data)

library(ROCit)

ROCit_obj <- rocit(score=exp,class=rta, negref=0)
plot(ROCit_obj)

summary(ROCit_obj)

detach(data)
```

En la función *rocit* se marca *negref* como 0 para indicar que en los datos la etiqueta 0 se corresponde con el valor de referencia, es decir, es el valor de la etiqueta para muestras negativas. Así se evita que la librería pueda tratar el test diagnóstico como un anti-marcador. 

Se obtiene la siguiente *auc* (0.725):

```r
Method used: empirical   
Number of positive(s): 20
Number of negative(s): 33
Area under curve: 0.725
```

Un *auc* por encima de 0.5 nos indica que el test diagnóstico es significativamente relevante a la hora de detectar los nódulos infectados. El *auc* representa la probabilidad de que dado un par de individuos, uno sano y otro enfermo, estos sean correctamente clasificados. En este caso habría una probabilidad de 72.5% de que este par de individuos fuesen clasificados correctamente.

El *auc* y el test U de Mann-whitney son equivalente. Si calculamos el test para estos datos obtenemos: 

```r
> wilcox.test(exp[rta==1], exp[rta==0])

	Wilcoxon rank sum test with continuity correction

data:  exp[rta == 1] and exp[rta == 0]
W = 478.5, p-value = 0.006569
alternative hypothesis: true location shift is not equal to 0
```

Con un p-valor menor de 0.01 lo que nos indica que existen diferencias significativas entre las funciones de distribución de los dos grupos diagnosticados.

Con la siguiente fórmula se obtiene el *auc* a partir del test U de Mann-whitney:

$$AUC=\frac{U}{n_1n_2}$$

Donde $n_1$ y $n_2$ son el número de observaciones para los grupos de positivos y negativos. Con el siguiente código se calcula el *auc* usando el test U de Mann-whitney:

```r
> U <- as.numeric(wilcox.test(exp[rta==1], exp[rta==0])$statistic)
> U/(sum(rta==0) * sum(rta==1))
[1] 0.725
```

Vemos que obtenemos el mismo valor 0.725.

 

A continuación vemos la curva *roc*:

![Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej1_roc_ROCit.png](Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej1_roc_ROCit.png)

Curva *roc* obtenida usando la librería *ROCit*

Vemos en la imagen que la curva *roc* se encuentra por encima de la diagonal en prácticamente todos los puntos indicando de nuevo que el modelo tiene capacidad predictiva. Usando esta librería se obtiene la gráfica con los ejes clásicos de fracción de verdaderos positivos (Sensibilidad) y fracción de falsos positivos (1-especificidad). Además esta librería nos indica automáticamente uno de los puntos de corte óptimos (Índice de Youden).

Tras el análisis realizado podemos concluir que se puede usar este biomarcador para detectar la presencia de nódulos infectados. Pero para esto es importante encontrar un punto de corte óptimo para conseguir maximizar la capacidad diagnóstica del biomarcador.

El punto de corte ideal sería aquel en el que la sensibilidad y la especificidad son máximo, pero esto nunca es posible. Existe una situación de compromiso entre estas dos medidas. Existen diferentes métodos para elegir buenos puntos de corte en los que se intenta maximizar los dos valores.

Con el siguiente código en R se calcula dos posibles puntos de corte óptimos. La esquina noroeste (método closest.topleft) y el Índice de Youden (maximiza la distancia con la diagonal de la curva *roc*):

```r
> library(pROC)
> roc_obj <- roc(rta, exp)
Setting levels: control = 0, case = 1
Setting direction: controls < cases
> coords(roc_obj, "best", transpose = FALSE , best.method="youden") 
1       665   0.7272727         0.8
> coords(roc_obj, "best",  transpose = FALSE, best.method="closest.topleft")
  threshold specificity sensitivity
1       665   0.7272727         0.8
```

Usando los dos métodos descritos se obtiene el mismo punto de corte (665). Esto es posible, ya que en general los dos métodos tienen el mismo objetivo (maximizar la sensibilidad y la especificidad) y pueden coincidir en el mismo punto los dos.

Este valor se interpreta como que para valores menores de 665 de la variable explicativa no hay nódulo infectado, y para valores mayores de 665 se predice que hay nódulo infectado. (La dirección se obtiene a partir de la respuesta en R: `Setting direction: controls < cases` que indica que los valores de control son los menores, y los valores de los casos (positivos) son los mayores.)

Estas dos técnicas nos indican puntos de cortes óptimos generalistas, teniendo en cuenta solo los datos estadísticos. Este debe ser un valor orientativo el cual puede ser actualizado teniendo en cuenta otra información y características del problema, como por ejemplo la prevalencia de la enfermedad y el objetivo de la predicción. 

Por ejemplo, puede ser inadmisible predecir nódulos infectados como no infectados (falsos negativos) porque la mortalidad es altísima si no se recibe tratamiento temprano. En este caso se podría admitir un número mayor de falsos positivos y sería conveniente disminuir el valor del punto de corte.

Al no tener una mayor información sobre el problema se ofrece como respuesta 665 como punto de corte obtenido por los métodos descritos anteriormente, prediciendo la presencia de nódulo infectado para valores mayores a 665.

### Ejercicio 2. (2 puntos)

**Se pide analizar los datos del fichero *le10.txt* alojado en el curso virtual mediante regresión logística, evaluando su comportamiento a través de la curva *roc*. ¿Cuál es la relación con el ejercicio anterior? ¿Se puede utilizar la regresión logística para predecir la presencia de nódulos infectados?**

Mediante el siguiente código en R se realiza un ajuste de regresión logística en los datos:

```r
rm(list=ls())

data = read.table('le10.txt', header = T)
attach(data)

glm.fits=glm(rta~exp, family=binomial)
summary (glm.fits)
```

Obteniéndose:

```r
Call:
glm(formula = rta ~ exp, family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.0144  -0.9120  -0.8165   1.2871   1.5971  

Coefficients:
             Estimate Std. Error z value Pr(>|z|)  
(Intercept) -1.927032   0.921039  -2.092   0.0364 *
exp          0.002040   0.001257   1.624   0.1045  
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 70.252  on 52  degrees of freedom
Residual deviance: 67.116  on 51  degrees of freedom
AIC: 71.116

Number of Fisher Scoring iterations: 4
```

Vemos que se obtiene un coeficiente de intercepción de -1.927 con significancia estadística (p_valor < 0.05), y un coeficiente para la variable explicativa de 0.002040 con un p_valor de 0.1045 por tanto no hay evidencias claras de una asociación real entre la variable respuesta y la variable explicativa usando el método de regresión logística, como primera aproximación.

El coeficiente de intercepción simplemente sirve para calcular la probabilidad cuando la variable explicativa vale 0, no ofrece ningún valor extra a la predicción.

El coeficiente de la variable explicativa al ser positivo nos indica que al aumentar el valor de esta variable aumentan las probabilidades de una respuesta positiva (1). Este resultado está en concordancia con lo obtenido en el ejercicio 1. Al estar utilizando un modelo de regresión logística no es trivial conocer el aumento en probabilidad al aumentar en un punto la variable explicativa. Este aumento depende del valor del que se parte. Podemos graficar la distribución de las predicciones para ver que ese aumento no es lineal:

```r
exp.new=data.frame(exp=seq(-500,2000,1))
preds = predict(glm.fits, exp.new, type = 'response')

plot(exp.new$exp, preds ,type = "l", col = "red", lwd=3, ylim=c(0,1))
lines(exp,rta, type = "p", col='blue', pch = 19)
```

![Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_todo.png](Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_todo.png)

En azul los datos de entrenamiento y en rojo la función de predicciones de la regresión logística

Podemos observar en la imagen que la pendiente de la curva cambia en cada valor de la variable explicativa.

A continuación calculamos la curva *roc* para este modelo:

```r
library(pROC)

roc_obj <- roc(rta, predict(glm.fits, type = 'response'))
plot(roc_obj, print.auc=TRUE)
coords(roc_obj, "best", transpose = FALSE , best.method="youden")
plot(roc_obj, print.auc=TRUE, print.thres = "best")
```

![Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_roc.png](Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_roc.png)

Curva roc para el modelo de regresión logística

Observamos que la curva *roc* es idéntica a la obtenida en el ejercicio 1, con el mismo *auc* (0.725). Además las coordenadas del punto de corte óptimo dado por el Índice de Youden son las mismas (0.727, 0.800). Podemos comprobar que el valor de probabilidad de 0.361 (el punto de corte) equivale al valor del punto de corte obtenido en el ejercicio 1(665):

```r
> threshold = data.frame(exp=c(665))
> predict(glm.fits, threshold, type = 'response')
        1 
0.3611488
```

Vemos que el punto 665 (el punto de corte calculado en el ejercicio 1) se corresponde con una probabilidad de 0.361, el punto de corte que hemos obtenido para la regresión logística.

En este caso valores de probabilidades menores de 0.361 corresponderían con la ausencia de nódulos infectados, y valores mayores de 0.361 con la presencia de nódulos infectados. Como se explicó en el ejercicio anterior el valor del punto de corte se puede ajustar teniendo en cuenta más información acerca del problema.

Podemos concluir que el método de la regresión logística es equivalente al análisis usado en el ejercicio uno, llegando a los mismos resultados. Los dos métodos predecirían de manera idéntica la presencia de nódulos infectados. Por tanto, siguiendo los mismos razonamientos que en el ejercicio anterior este método es adecuado para predecir nódulos infectados.

Como último punto, se ha realizado el mismo estudio, pero eliminando el último dato, el cual tenía un valor muy alto para la variable explicativa y sin embargo, no indicaba la presencia de nódulo infectado. Por tanto, este punto podría ser un valor atípico.

```r
rm(list=ls())

data = read.table('le10_2.txt', header = T)
attach(data)

glm.fits=glm(rta~exp, family=binomial)
summary (glm.fits)

exp.new=data.frame(exp=seq(-500,2000,1))
preds = predict(glm.fits, exp.new, type = 'response')

plot(exp.new$exp, preds ,type = "l", col = "red", lwd=3, ylim=c(0,1))
lines(exp,rta, type = "p", col='blue', pch = 19)
```

```r
Call:
glm(formula = rta ~ exp, family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.6828  -0.8437  -0.6700   1.1000   1.8017  

Coefficients:
             Estimate Std. Error z value Pr(>|z|)   
(Intercept) -3.662050   1.268951  -2.886  0.00390 **
exp          0.004706   0.001819   2.587  0.00968 **
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 69.293  on 51  degrees of freedom
Residual deviance: 60.452  on 50  degrees of freedom
AIC: 64.452

Number of Fisher Scoring iterations: 4
```

Como principal diferencia vemos que ahora el coeficiente de la variable explicativa si es estadísticamente diferente de cero con un valor de 0.004706 y un p-valor de 0.00968 <0.01.

![Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_impr.png](Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_impr.png)

En azul los datos de entrenamiento y en rojo la función de predicciones de la regresión logística

Podemos observar en la gráfica que ahora la función tiene una forma de sigmoide más pronunciada.

Si calculamos el punto de corte óptimo mediante la curva *roc* obtenemos:

```r
library(pROC)

roc_obj <- roc(rta, predict(glm.fits, type = 'response'))
plot(roc_obj, print.auc=TRUE)
coords(roc_obj, "best", transpose = FALSE , best.method="youden")
plot(roc_obj, print.auc=TRUE, print.thres = "best")
```

![Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_roc_2.png](Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej2_roc_2.png)

Curva roc para el modelo de regresión logística

Vemos que la curva *roc* es muy similar a la obtenida con todos los datos, y el punto de corte aumenta a 0.370 (antes era 0.361). El *auc* también aumenta a 0.748.

Podemos concluir que la exclusión de ese punto mejora el ajuste logístico, haciendo que el coeficiente de la variable explicativa sea significativamente diferente de cero. Eliminando ese punto, podemos asegurar que los datos permiten predecir la presencia de nódulos y que el ajuste no es una mera casualidad estadística. (En el ejercicio 1 también se estudió mediante el test U de Mann-whitney que los grupos de casos y control tienen distribuciones diferentes, por tanto son separables.)

### Ejercicio 3. (2 puntos)

**Se seleccionan 100 individuos enfermos (rta=1) y 100 individuos sanos (rta=0). A estos 200 individuos se les aplica una prueba diagnóstica con dos resultados posibles (exp=1: positivo, exp=0: negativo) y se obtienen los datos del fichero *d_d_5.txt*. Adicionalmente, se estima que la prevalencia de la enfermedad es del 15%. Utilizando regresión logística, ¿se puede estimar la probabilidad de que esté enfermo un individuo con resultado positivo?**

Podemos empezar aplicando la regresión logística a los datos tal y como los hemos obtenido:

```r
rm(list=ls())

evaluate = function(coeff) {
  sol = exp(coeff[1]+coeff[2]*1)/(1+exp(coeff[1]+coeff[2]*1))
  names(sol) = NULL
  return(sol)
}

#fit sin ajustar la prevalencia
data = read.table('d_d_5.txt', header = T)
attach(data)
glm.fits=glm(rta~exp, family=binomial)
summary (glm.fits)
evaluate(glm.fits$coefficients)
```

Se ha creado una función `evaluate` que calcula la probabilidad que pide el ejercicio usando los coeficientes de la regresión logística.

Se obtienen los siguientes resultados:

```r
> summary (glm.fits)

Call:
glm(formula = rta ~ exp, family = binomial)

Deviance Residuals: 
   Min      1Q  Median      3Q     Max  
-2.797  -0.201   0.000   0.201   2.797  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  -3.8918     0.7142  -5.449 5.07e-08 ***
exp           7.7836     1.0101   7.706 1.30e-14 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 277.259  on 199  degrees of freedom
Residual deviance:  39.216  on 198  degrees of freedom
AIC: 43.216

Number of Fisher Scoring iterations: 6

> evaluate(glm.fits$coefficients)
[1] 0.98
```

Observamos que el ajuste es bueno, obteniéndose p-valores muy bajos para el intercepto y la variable explicativa. 

Con la función `evaluate` obtenemos que la probabilidad de estar enfermo con un resultado positivo es de 0.98. Sin embargo, este valor no es correcto porque la prevalencia en nuestros datos no es la misma que la prevalencia de la población. En nuestros datos la prevalencia es 0.5 (según el enunciado tenemos 100 individuos enfermos y 100 sanos) y la prevalencia de la población es 0.15.

Una primera aproximación sería construir un nuevo data set en el cual la prevalencia sea 0.15, pero se mantenga las proporciones, es decir la sensibilidad y la especificidad no cambian. Mediante el siguiente código en R se simula ese dataset:

```r
#fit dataset con la prevalencia ajustada
rta2 = c(rep(c(1),each=100),rep(c(0),each=567))
exp2 = c(rep(c(0),each=2),rep(c(1),each=110),rep(c(0),each=555))
data2 = data.frame(rta2,exp2)
glm.fits2=glm(data2$rta2~data2$exp2, family=binomial)
summary (glm.fits2)
evaluate(glm.fits2$coefficients)
```

Se obtienen los siguientes resultados:

```r
> summary (glm.fits2)

Call:
glm(formula = data2$rta2 ~ data2$exp2, family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.1050  -0.0848  -0.0848  -0.0848   3.3554  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  -5.6258     0.7084  -7.942 1.99e-15 ***
data2$exp2    7.7259     0.7716  10.013  < 2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 563.72  on 666  degrees of freedom
Residual deviance: 102.32  on 665  degrees of freedom
AIC: 106.32

Number of Fisher Scoring iterations: 8

> evaluate(glm.fits2$coefficients)
[1] 0.8909091
```

Observamos que el ajuste sigue siendo bueno y que los parámetros del ajuste han cambiado ligeramente, en especial para el intercepto que disminuye hasta -5.6258.

Se obtiene en este caso una probabilidad de 0.8909. Este valor se debe acercar más al valor real. Es mejor que en el caso anterior como era de esperar, ya que al disminuir la prevalencia hay más posibilidades de obtener falsos positivos. Sin embargo, esta aproximación no es exacta porque al generar el dataset se deben realizar aproximaciones, ya que no puede haber fracciones de individuos enfermos. Por tanto los valores de sensibilidad y especificidad cambian ligeramente. Se necesitaría un dataset muy grande para mejorar la aproximación usando esta técnica. Aumentar el tamaño del dataset es contraproducente, ya que añadiría complejidad al cálculo de la regresión logística.

Este tipo de problema se corresponde con un case-control study. En el cual se han preseleccionado un número arbitrario de casos y de controles. Normalmente en estos estudios para simplificar la toma de datos, y en general, el tamaño de los datos, la prevalencia en el dataset es diferente a la de la población. La regresión logística se emplea a menudo en este tipo de estudios porque si se conoce la prevalencia de la población es fácil actualizar los parámetros del modelo para que se ajuste a las probabilidades de la población.

En bibliografía se encuentra información de que solo es necesario actualizar el valor de $\beta_0$, mientras que el resto de parámetros permanecen invariantes. En un video de los autores del libro de referencia de la asignatura, se explica este tipo de problemas e indican la siguiente fórmula para actualizar el valor de $\beta_0$:

$$\hat\beta_0^* = \hat\beta_0 + log\frac{\pi}{1-\pi}-log\frac{\tilde\pi}{1-\tilde\pi}$$

[StatsLearning Lect6d 110613](https://www.youtube.com/watch?v=GavRXXEHGqU)

Siendo $\pi$ la prevalencia de la población y $\tilde\pi$ la prevalencia de la muestra. Con el siguiente código en R se actualiza el valor de $\beta_0$ y se calcula la probabilidad con el parámetro actualizado:

```r
#actualizando beta_0
pi = 0.50
pi_real = 0.15
#parametros obtenidos con la regresion logistica de la muestra
b0 = -3.8918
b1 = 7.7836
b0_real = b0 + log(pi_real/(1-pi_real)) - log(pi/(1-pi))

evaluate(c(b0_real,b1))
```

Se obtienen los siguientes resultados:

```r
> b0_real
[1] -5.626401
> evaluate(c(b0_real,b1))
[1] 0.8963396
```

Vemos que el parámetro $\beta_0$ pasa a valer de -3.89 a -5.62. Un valor muy próximo al obtenido en la regresión logística con el dataset simulado.

Obtenemos una probabilidad de 0.8963 parecida también a la obtenida en la regresión logística simulada.

Con estos últimos cálculos podemos concluir que se puede estimar la probabilidad de que esté enfermo un individuo con resultado positivo mediante regresión logística. El valor es de 0.8963 y se debe resaltar que es importante corregir los valores obtenidos con el valor real de la prevalencia.

Como último punto se ha calculado teóricamente cuáles serían los parámetros de la regresión logística teniendo en cuenta desde el principio la prevalencia poblacional, con el objetivo de estar seguros de que el parámetro $\beta_1$ no cambia.

Se tiene que:

$$ln(\frac{p(x)}{1-p(x)}) = \beta_0 + \beta_1x$$

Cuando $x = 0$:

$$ln(\frac{P(y=1|x=0)}{1-P(y=1|x=0)}) = \beta_0$$

$$P(y=1|x=0) = \frac{P(x=0|y=1)P(y=1)}{P(x=0|y=1)P(y=1) + P(x=0|y=0)P(y=0)} = \frac{(1-se)prev}{(1-se)prev+ es(1-prev)}$$

Conociendo la sensibilidad, la especificidad y la prevalencia se puede calcular $\beta_0$. 

Por otro lado, cuando $x = 1$:

$$ln(\frac{P(y=1|x=1)}{1-P(y=1|x=1)}) = \beta_0+\beta_1$$

$$P(y=1|x=1) = \frac{P(x=1|y=1)P(y=1)}{P(x=1|y=1)P(y=1) + P(x=1|y=0)P(y=0)} = \frac{se*prev}{se*prev+ (1-es)(1-prev)}$$

Con la fórmula anterior, trás calcular $\beta_0$ se puede obtener el valor de $\beta_1$. 

El siguiente código en R realiza estos cálculos y obtiene el valor de la probabilidad:

```r
#calculo betas teorico
cm = confusionMatrix(table(rta,exp))
se = cm$byClass[1] #sensibilidad
names(se) = NULL
es = cm$byClass[2] #especificidad
names(es) = NULL
prev = 0.15 #prevalencia
P1_0 = ((1-se)*prev)/((1-se)*prev+es*(1-prev)) #P(y=1|x=0)
b0_t = log(P1_0/(1-P1_0)) #beta_0 teorico
P1_1 = (se*prev)/(se*prev+(1-es)*(1-prev)) #P(y=1|x=1)
b1_t = log(P1_1/(1-P1_1))-b0_t #beta_1 teorico
b0_t
b1_t
evaluate(c(b0_t,b1_t))
```

Se obtienen los siguientes resultados:

```r
> b0_t
[1] -5.626421
> b1_t
[1] 7.783641
> evaluate(c(b0_t,b1_t))
[1] 0.8963415
```

Primero, vemos que obtenemos el mismo $\beta_0$ que usando la fórmula proporcionada por los autores del libro para actualizar el valor teniendo en cuenta la prevalencia. 

Por otro lado, el valor de $\beta_1$ no cambia con respecto al modelo de regresión logística antes de tener en cuenta la prevalencia.

Obteniendo por tanto una probabilidad de 0.8963 de estar enfermo al dar positivo en el test. La misma probabilidad que en el apartado anterior. Esto justifica que la prevalencia solo afecta al valor de $\beta_0$, pero no cambia los valores del resto de parámetros. Explicando que en la literatura solo se encuentran fórmulas para actualizar $\beta_0$. 

Sin embargo, hay que tener en cuenta que en la realización de este ejercicio no se ha realizado una demostración teórica. Solo puedo asegurar que en este caso $\beta_1$ no cambia. 

### Ejercicio 4. (CALC) (1 punto)

**En el contexto del modelo de regresión logística sin ninguna variable explicativa, se pide demostrar una de las siguientes relaciones equivalentes:**

$$\ell(\beta_0)= (\frac {1}{1+e^{-\beta_0}})^{r_1} (1-\frac {1}{1+e^{-\beta_0}})^{r_0}$$

$$\ell(\beta_0)= \frac {e^{-r_0\beta_0}}{(1+e^{-\beta_0})^{r_1+r_0}}$$

**donde $r_1$ es el número de individuos con $Y = 1$ y $r_0$ es el número de individuos con $Y = 0$**

Para resolver este ejercicio partimos de las siguientes funciones:

L**ogistic function**

$$p(x) = \frac {e^{\beta_0+\beta_1X_1+...+\beta_pX_p}}{1+e^{\beta_0+\beta_1X_1+...+\beta_pX_p}}$$

**Likelihood function**

$$\ell(\beta_0,\beta_1,...)= \prod_{i:y_i=1} p(x_i) \prod_{i':y_{i'}=0} (1-p(x_{i'}) )$$

Primero, partiendo de la función logística nos quedamos solo con $\beta_0$, ya que al no tener variables explicativas el resto de términos son cero:

$$p(x) = \frac {e^{\beta_0}}{1+e^{\beta_0}}$$

Dividimos numerador y denominador por $e^{\beta_0}$:

$$p(x) = \frac {e^{\beta_0}}{1+e^{\beta_0}} = \frac {1}{1/e^{\beta_0} + 1} = \frac {1}{1 + e^{-\beta_0}}$$

Sustituimos $p(x)$ en la likelihood function:

$$\ell(\beta_0)= \prod_{i:y_i=1} \frac {1}{1 + e^{-\beta_0}} \prod_{i':y_{i'}=0} (1-\frac {1}{1 + e^{-\beta_0}}) )$$

El primer productorio es  $r_1$ veces el mismo término y el segundo es $r_0$ veces, por tanto tenemos:

$$\ell(\beta_0)=(\frac {1}{1 + e^{-\beta_0}})^{r_1}  (1-\frac {1}{1 + e^{-\beta_0}})^{r_0}$$

Por tanto, hemos demostrado la primera de las expresiones.

### Ejercicio 5. (CALC) (1 punto)

**En el contexto del modelo de regresión logística sin ninguna variable explicativa, se pide demostrar una de las siguientes relaciones equivalentes:**

$$\frac{\partial}{\partial\beta_0} Ln\ell(\beta_0) = \frac {r_1e^{-\beta_0}-r_0}{1+e^{-\beta_0}}$$

$$\frac{\partial}{\partial\beta_0} Ln\ell(\beta_0) = - r_0 + (r_0+r_1)\frac{e^{-\beta_0}}{1+e^{-\beta_0}}$$

**donde $r_1$ es el número de individuos con $Y = 1$ y $r_0$ es el número de individuos con $Y = 0$**

Partimos de la expresión demostrada en el ejercicio anterior:

$$\ell(\beta_0)=(\frac {1}{1 + e^{-\beta_0}})^{r_1}  (1-\frac {1}{1 + e^{-\beta_0}})^{r_0}$$

Aplicamos logaritmo natural:

$$Ln\ell(\beta_0)=Ln[(\frac {1}{1 + e^{-\beta_0}})^{r_1}  (1-\frac {1}{1 + e^{-\beta_0}})^{r_0}]=$$

Aplicando propiedades de los logaritmos:

$$=Ln[(\frac {1}{1 + e^{-\beta_0}})^{r_1}] + Ln[(1-\frac {1}{1 + e^{-\beta_0}})^{r_0}]=$$

$$=r_1Ln(\frac {1}{1 + e^{-\beta_0}}) + r_0Ln(1-\frac {1}{1 + e^{-\beta_0}})=$$

$$=r_1Ln(1)-r_1Ln(1 + e^{-\beta_0}) + r_0Ln(\frac {e^{-\beta_0}}{1 + e^{-\beta_0}})=$$

$$=-r_1Ln(1 + e^{-\beta_0}) + r_0Ln(e^{-\beta_0})-r_0Ln(1 + e^{-\beta_0})=$$

$$=-(r_0+r_1)Ln(1 + e^{-\beta_0}) + r_0Ln(e^{-\beta_0})=$$

$$=-(r_0+r_1)Ln(1 + e^{-\beta_0}) - r_0\beta_0 = - r_0\beta_0 -(r_0+r_1)Ln(1 + e^{-\beta_0})$$

Derivando la expresión anterior:

$$\frac{\partial}{\partial\beta_0} Ln\ell(\beta_0) = \frac{\partial}{\partial\beta_0}[- r_0\beta_0 -(r_0+r_1)Ln(1 + e^{-\beta_0})] = $$

$$= - r_0\frac{\partial}{\partial\beta_0}[\beta_0] - (r_0+r_1)\frac{\partial}{\partial\beta_0}[Ln(1 + e^{-\beta_0})] = $$

$$= - r_0 - (r_0+r_1)\frac{1}{(1 + e^{-\beta_0})}e^{-\beta_0}(-1) = $$

$$= - r_0 + (r_0+r_1)\frac{e^{-\beta_0}}{(1 + e^{-\beta_0})} $$

Por tanto, hemos demostrado la segunda expresión.

### Ejercicio 6. (2 puntos)

**Se ha realizado un estudio para averiguar si el hecho de que un estudiante apruebe o no un examen final (examen=1: S, examen=0: No) depende de:** 

- **Ser repetidor (repetidor=1: S, repetidor=0: No).**
- **Tipo de vídeos que recibe en el material docente
(video=0: modalidad 0, video=1: modalidad 1, video=2: modalidad 2).**
- **Nota obtenida en la práctica 1 (practica1).**
- **Nota obtenida en la práctica 2 (practica2).**
- **Nota obtenida en la práctica 3 (practica3).**
- **Nota obtenida en un test online de 80 preguntas (online).**

**Han participado en el estudio 41 estudiantes. Los datos experimentales están en el fichero *d_dncccc_2.txt* alojado en el curso virtual. Se ha decidido crear las variables centradas *practica1c*, *practica2c*, *practica3c* y *onlinec* a partir de sus correspondientes *practica1*, *practica2*, *practica3* y *online*. Se pide:**

- **En un modelo de regresión logística simple, ¿qué variables influyen en la respuesta? Interpretar los coeficientes.**

Código para obtener las diferentes regresiones logísticas simples:

```r
rm(list=ls())

data = read.table('d_dncccc_2.txt', header = T)
attach(data)

practica1c = practica1 - mean (practica1) # Centrado de practica1
practica2c = practica2 - mean (practica2) # Centrado de practica2
practica3c = practica3 - mean (practica3) # Centrado de practica3
onlinec = online - mean (online) # Centrado de online

videod0 = 1*( video == 0) # Variable dummy video tipo 0
videod1 = 1*( video == 1) # Variable dummy video tipo 1
videod2 = 1*( video == 2) # Variable dummy video tipo 2

n_data = data.frame(repetidor,practica1c,practica2c,practica3c,onlinec,videod0,videod1,videod2)

for (f in c(1:8)){
  print(names(n_data[f]))
  print(summary(glm(examen~unlist(n_data[f]), family=binomial)))
}
```

En la primera parte del código se realiza el centrado de las diferentes variables. Además se crean tres variables dummies para codificar la variable video en cada una de sus clases. Se crea una variable dummy para cada clase para estudiarlas independientemente con un modelo cada una.

Resultados:

```r
[1] "repetidor"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.7712  -1.1278   0.6835   0.6835   1.2278  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)  
(Intercept)        -0.1178     0.4859  -0.242   0.8085  
unlist(n_data[f])   1.4528     0.6991   2.078   0.0377 *
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 48.072  on 39  degrees of freedom
AIC: 52.072

Number of Fisher Scoring iterations: 4

[1] "practica1c"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.5580  -1.3740   0.8397   0.8990   1.0917  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)  
(Intercept)         0.6632     0.3316   2.000   0.0455 *
unlist(n_data[f])   0.1641     0.2495   0.657   0.5109  
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 52.215  on 39  degrees of freedom
AIC: 56.215

Number of Fisher Scoring iterations: 4

[1] "practica2c"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.0004  -0.8980   0.5391   0.5391   1.6134  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)   
(Intercept)         0.8374     0.4008   2.089  0.03669 * 
unlist(n_data[f])   0.2839     0.0922   3.079  0.00207 **
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 41.152  on 39  degrees of freedom
AIC: 45.152

Number of Fisher Scoring iterations: 4

[1] "practica3c"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.0731  -0.9349   0.5361   0.6044   1.5915  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)   
(Intercept)         0.8167     0.3918   2.085  0.03708 * 
unlist(n_data[f])   0.4640     0.1583   2.932  0.00337 **
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 42.315  on 39  degrees of freedom
AIC: 46.315

Number of Fisher Scoring iterations: 4

[1] "onlinec"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.1560  -1.0276   0.4143   0.8668   1.3774  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)  
(Intercept)        1.00957    0.44940   2.246   0.0247 *
unlist(n_data[f])  0.09571    0.04369   2.190   0.0285 *
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 44.476  on 39  degrees of freedom
AIC: 48.476

Number of Fisher Scoring iterations: 5

[1] "videod0"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.9728  -1.2735   0.5553   1.0842   1.0842  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)  
(Intercept)         0.2231     0.3873   0.576    0.565  
unlist(n_data[f])   1.5686     0.8563   1.832    0.067 .
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 48.579  on 39  degrees of freedom
AIC: 52.579

Number of Fisher Scoring iterations: 4

[1] "videod1"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.8562  -0.8576   0.6272   0.6272   1.5353  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)   
(Intercept)         1.5261     0.4934   3.093  0.00198 **
unlist(n_data[f])  -2.3370     0.7776  -3.006  0.00265 **
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 42.325  on 39  degrees of freedom
AIC: 46.325

Number of Fisher Scoring iterations: 4

[1] "videod2"

Call:
glm(formula = examen ~ unlist(n_data[f]), family = binomial)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.7552  -1.3401   0.6945   1.0230   1.0230  

Coefficients:
                  Estimate Std. Error z value Pr(>|z|)
(Intercept)         0.3747     0.3917   0.957    0.339
unlist(n_data[f])   0.9246     0.7600   1.217    0.224

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 51.047  on 39  degrees of freedom
AIC: 55.047

Number of Fisher Scoring iterations: 4
```

De entre todos los modelos estudiados solo algunos de ellos presentan un p-valor < 0.05 para la variable estudiada. En estos casos, se considera que existe una relación significativa entre la variable estudiada y la variable objetivo examen. Estas variables significativas son: **repetidor, practica2c, practica3c, onlinec, videod1**.

Analizando los coeficientes podemos observar que todas las variables, menos videod1, tienen un coeficiente positivo, lo que indica que la presencia o aumento en esas variables aumenta la probabilidad de aprobar el examen. Por otro lado para videod1 el coeficiente es negativo, por tanto, este tipo de videos disminuye la probabilidad de aprobar el examen.

No podemos saber el cambio en la probabilidad por el cambio por unidad de los diferentes parámetros porque en la regresión logística la magnitud del cambio depende del valor de partida. Solo se puede estimar la dirección del cambio como se ha hecho en el párrafo anterior.

- **En un modelo de regresión logística múltiple con todas las variables explicativas, ¿qué sucede?, ¿es fiable este modelo?**

Con el siguiente código estudiamos el modelo que incluye todas las variables. Quitamos una de las variables dummies de video, ya que la información de la tercera se encuentra codificada en la negación de las otras dos. También se utiliza la función vif para estudiar la multicolinealidad.

```r
model = glm(examen~ . -videod0, data = n_data, family=binomial)
summary(model)
library (car)
vif(model)
```

Estos son los resultados obtenidos:

```r
> summary(model)

Call:
glm(formula = examen ~ . - videod0, family = binomial, data = n_data)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.4756  -0.4553   0.3010   0.5147   1.5643  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)  
(Intercept)  1.49118    1.33699   1.115   0.2647  
repetidor    0.85506    1.09852   0.778   0.4363  
practica1c   0.04997    0.86418   0.058   0.9539  
practica2c   0.36523    0.80814   0.452   0.6513  
practica3c  -0.27562    1.63802  -0.168   0.8664  
onlinec      0.02764    0.05671   0.487   0.6260  
videod1     -2.44455    1.24965  -1.956   0.0504 .
videod2     -0.43001    1.33677  -0.322   0.7477  
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 31.824  on 33  degrees of freedom
AIC: 47.824

Number of Fisher Scoring iterations: 5

> library (car)
> vif(model)
 repetidor practica1c practica2c practica3c    onlinec    videod1    videod2 
  1.508093   6.343349  58.302318  80.656627   2.507662   1.889500   1.960018
```

Vemos que el modelo con todas las variables no presenta ningún coeficiente que se pueda considerar estadísticamente diferente de cero (tenemos p-valor > 0.05). El único coeficiente que está muy cerca del límite de 0.05 es el de la variable videod1 con un p-valor de 0.0504.

Observamos que el error estándar es bastante amplio en la mayoría de los casos pudiendo influir en los altos valores de p. Una de las posibles causas de eso es la multicolinealidad.

 Con la función vif se han calculado los variance inflation factor que dan una indicación del nivel de colinealidad para las diferentes variables. Un valor de 1 indica la ausencia de colinealidad y valores altos la presencia de colinealidad. Con valores mayores de 5 o 10 es un claro indicativo de un problema de colinealidad. En este caso tenemos valores altos para las variables practica1c, practica2c y practica3c. Por tanto debe existir colinealidad entre estas variables.

Podemos considerar que este modelo no es fiable porque ninguna variable se puede considerar significativamente influyente y por la presencia de colinealidad. 

- **Obtener el mejor modelo de regresión logística por pasos. ¿Qué sucede?**

Para mejorar el modelo se han seleccionado las variables más influyentes usando la función stepAIC de la libreria MASS. Esta función realiza una selección por pasos de las variables, usando como puntuación el valor de AIC de cada modelo. 

En la función se ha usado el argumento `direction ='both'`. Esto sirve para que la función compruebe tanto los modelos con menos variables como los modelos con más. Esta técnica puede servir para añadir una variable que fue eliminada inicialmente (por ejemplo por colinealidad) pero que cuando quedan menos variables al ser añadida mejora el modelo. Esto aumenta la complejidad computacional de la técnica, pero al tener pocas variables en este caso nos lo podemos permitir. 

```r
library(MASS)

temp = stepAIC(model,direction ='both')
summary(temp)
```

Resultados obtenidos:

```r
Start:  AIC=47.82
examen ~ (repetidor + practica1c + practica2c + practica3c + 
    onlinec + videod0 + videod1 + videod2) - videod0

             Df Deviance    AIC
- practica1c  1   31.828 45.828
- practica3c  1   31.853 45.853
- videod2     1   31.928 45.928
- practica2c  1   32.033 46.033
- onlinec     1   32.110 46.110
- repetidor   1   32.438 46.438
<none>            31.824 47.824
- videod1     1   36.108 50.108

Step:  AIC=45.83
examen ~ repetidor + practica2c + practica3c + onlinec + videod1 + 
    videod2

             Df Deviance    AIC
- practica3c  1   31.884 43.884
- videod2     1   31.939 43.939
- onlinec     1   32.110 44.110
- practica2c  1   32.324 44.324
- repetidor   1   32.525 44.525
<none>            31.828 45.828
+ practica1c  1   31.824 47.824
- videod1     1   36.114 48.114

Step:  AIC=43.88
examen ~ repetidor + practica2c + onlinec + videod1 + videod2

             Df Deviance    AIC
- videod2     1   31.964 41.964
- onlinec     1   32.140 42.140
- repetidor   1   32.685 42.685
- practica2c  1   33.664 43.664
<none>            31.884 43.884
+ practica3c  1   31.828 45.828
+ practica1c  1   31.853 45.853
- videod1     1   36.123 46.123

Step:  AIC=41.96
examen ~ repetidor + practica2c + onlinec + videod1

             Df Deviance    AIC
- onlinec     1   32.211 40.211
- repetidor   1   33.164 41.164
- practica2c  1   33.681 41.681
<none>            31.964 41.964
+ videod2     1   31.884 43.884
+ practica3c  1   31.939 43.939
+ practica1c  1   31.955 43.955
- videod1     1   37.952 45.952

Step:  AIC=40.21
examen ~ repetidor + practica2c + videod1

             Df Deviance    AIC
- repetidor   1   33.309 39.309
<none>            32.211 40.211
+ onlinec     1   31.964 41.964
+ videod2     1   32.140 42.140
+ practica3c  1   32.201 42.201
+ practica1c  1   32.206 42.206
- videod1     1   38.496 44.496
- practica2c  1   40.194 46.194

Step:  AIC=39.31
examen ~ practica2c + videod1

             Df Deviance    AIC
<none>            33.309 39.309
+ repetidor   1   32.211 40.211
+ videod2     1   32.850 40.850
+ onlinec     1   33.164 41.164
+ practica1c  1   33.169 41.169
+ practica3c  1   33.255 41.255
- videod1     1   41.152 45.152
- practica2c  1   42.325 46.325
> summary(temp)

Call:
glm(formula = examen ~ practica2c + videod1, family = binomial, 
    data = n_data)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.3974  -0.4535   0.3410   0.5957   1.2313  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)   
(Intercept)   1.7610     0.6130   2.873  0.00407 **
practica2c    0.2942     0.1096   2.684  0.00727 **
videod1      -2.3909     0.9306  -2.569  0.01019 * 
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 33.309  on 38  degrees of freedom
AIC: 39.309

Number of Fisher Scoring iterations: 5
```

Vemos como en cada paso la función calcula el valor de AIC para cada modelo añadiendo o eliminando cada variable. Finalmente termina de iterar cuando no se mejora el modelo ni eliminando ni añadiendo ninguna variable.

El mejor modelo tiene solo las variables practica2c y videod1. Las dos son estadísticamente significantes con p-valores < 0.05. Por un lado practica2c tiene una correlación positiva con la variable examen, sin embargo videod1 presenta una correlación negativa.

Comparando cada coeficiente con los obtenidos en los modelos de regresión logística simple estos son muy simulares para las dos variables. Tienen el mismo signo, y una magnitud parecida en los dos casos. Por tanto parece indicar que no hay interacción entre la dos variables.

- **En el modelo anterior, ¿hay *confusión*?, ¿hay *interacción*?**

En una primera aproximación parece que no hay interacción entre las dos variables, ya que tiene valores muy parecidos a los que tenían en la regresión logística simple con solo una variable. A continuación podemos ver las características del modelo incluyendo el término de interacción:

```r
> int = glm(formula = examen ~ practica2c*videod1, family = binomial, 
+     data = n_data)
Warning message:
glm.fit: fitted probabilities numerically 0 or 1 occurred 
> summary(int)

Call:
glm(formula = examen ~ practica2c * videod1, family = binomial, 
    data = n_data)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-2.1143   0.0000   0.4757   0.4757   0.9685  

Coefficients:
                    Estimate Std. Error z value Pr(>|z|)   
(Intercept)           1.5453     0.5249   2.944  0.00324 **
practica2c            0.1609     0.1173   1.372  0.17011   
videod1             -27.2044  4945.2641  -0.006  0.99561   
practica2c:videod1   12.3896  2096.2252   0.006  0.99528   
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 52.644  on 40  degrees of freedom
Residual deviance: 24.366  on 37  degrees of freedom
AIC: 32.366

Number of Fisher Scoring iterations: 20
```

Vemos que incluyendo el término de la interacción se pierde la influencia de las variables, todas tienen p-valores > 0.05. Además los valores de error estándar crecen mucho para la variable videod1 y la interacción. Se está produciendo multicolinealidad. Por tanto, podemos concluir que no hay interacción entre las variables y que las dos afecta el valor de la respuesta de manera independiente.

Sobre la confusión podemos concluir de igual manera. Las variables tienen coeficientes con valores muy parecidos a los obtenidos en los modelos simples. Lo cual nos indica que los variables independientes y los valores que toman no están relacionados entre sí. Por tanto no se produce confusión.

- **¿Cuál es la curva *roc* del modelo final? ¿Cuál es su *auc*? ¿Cuál es el punto de corte óptimo para utilizar el modelo?**

Con el siguiente código calculamos la curva *roc*:

```r
library(pROC)

roc_obj = roc(examen, predict(temp, type = 'response'))
coords(roc_obj, "best", transpose = FALSE , best.method="youden")
plot(roc_obj, print.auc=TRUE, print.thres = "best")
```

Nos devuelve la siguiente gráfica:

![Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej6_roc.png](Modelado%20estadi%CC%81stico%20de%20datos%20Pra%CC%81ctica%202%20eaa1a71da8e247e193e2bd81376954cf/ej6_roc.png)

En la imagen podemos ver la curva *roc* del modelo final. Tiene un *auc* de 0.860 lo cual nos indica que es un buen modelo, que funciona mejor que un clasificador aleatorio (auc de 0.5).

El punto de corte óptimo calculado por el índice de Youden es 0.428. 

- **Para analizar estos datos, ¿se puede utilizar alguna otra técnica de las tratadas en esta asignatura? ¿Y alguna otra no tratada en esta asignatura?**

Para estudiar este tipo de datos se podría utilizar el análisis discriminante si se cumplen las condiciones de aplicabilidad.

Otras técnicas posibles son k-neighbourhood classifier, decision tree classifier, random forest classifier, naive bayes classifier o support vector machines, entre otros algoritmos de clasificación.