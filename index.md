<img src="Logo_UCO.png" style="width:5.0%" /> Genomics of drug
sensitivity in cancer (GDSC)
<img src="Logo_Facultad_de_Ciencias.png" style="width:7.5%" />

================
Cristina Gómez Baena, Alma Gavilán Velasco, Reyes Mª Jurado Jiménez
2025-12-15

# Introducción

La **Genomics of Drug Sensitivity in Cancer (GDSC)** es actualmente la
base de datos pública más amplia dedicada al estudio de la sensibilidad
a fármacos en cáncer.

La importancia de GDSC radica en que gracias a ella podemos identificar
biomarcadores moleculares que serán capaces de predecir la sensibilidad
a fármacos, lo cual es necesario para avanzar en la medicina referente
al cáncer. Las alteraciones genómicas condicionan profundamente la
respuesta terapéutica, por lo que recursos como GDSC permiten descubrir
relaciones funcionales entre mutaciones, vías biológicas y eficacia de
fármacos ([Yang et al. 2012](#ref-yang2012genomics)).

En este informe, utilizamos los datos almacenados en la [página del
proyecto GDSC](https://www.cancerrxgene.org/downloads/bulk_download). En
primer lugar, cargamos la librería
<span style="color:purple;font-family:Courier New;">readxl</span> y los
datos en el environment, creando un objeto llamado
<span style="color:purple;font-family:Courier New;">Data</span>.

La base de datos incluye 19 variables. Contabilizamos las variables
usando `ncol(Data)`. En nuestro proyecto seleccionaremos algunas de
estas y analizaremos relaciones entre ellas con la finalidad de evaluar
diferencias de sensibilidad entre distintos grupos comparables. Para
ello, haremos uso de varios test estadísticos (*t de student*, *chi
cuadrado*, *ANOVA*).

# Objetivos

## Objetivo general

Identificar patrones de respuesta a fármacos anticancerígenos en líneas
celulares del GDSC utilizando métricas relativas a la sensibilidad en
relación a rutas metabólicas y dianas moleculares.

## Objetivos específicos

- Comparar la sensibilidad a dos fármacos distintos que actúan sobre la
  misma vía metabólica (señalización de MAPK).
- Clasificar las rutas metabólicas en función a la sensibilidad que
  presentan a fármacos anticancerígenos.
- Identificar las dianas celulares que mejor responden a tratamiento con
  fármacos anticancerígenos.

# Hipótesis

H1.0: *Las medidas de sensibilidad entre los fármacos que actúan sobre
la señalización de p38 y JNK no difieren.*

H1.1: *Las medidas de sensibilidad entre los fármacos que actúan sobre
la señalización de p38 y JNK son diferentes.*

H2.0: *Distintas rutas metabólicas no presentan distintos niveles de
sensibilidad a fármacos anticancerígenos.*

H2.1: *Distintas rutas metabólicas presentan distintos niveles de
sensibilidad a fármacos anticancerígenos.*

H3.0: *La respuesta a los fármacos no varía entre distinas dianas
celulares.*

H3.1: *La respuesta a los fármacos varía según las dianas celulares.*

# Materiales y métodos

Para realizar nuestros datos, utilizamos la R version 4.4.3 (2025-02-28
ucrt).

Además, utilizaremos las librerías
<span style="color:purple;font-family:Courier New;">readxl</span>
cargada anteriormente ([Wickham and Bryan 2023](#ref-R-readxl)),
<span style="color:purple;font-family:Courier New;">dplyr</span>
([Wickham et al. 2023](#ref-R-dplyr)),
<span style="color:purple;font-family:Courier New;">knitr</span> ([Xie
2023](#ref-R-knitr), [2015](#ref-knitr2015), [2014](#ref-knitr2014)),
<span style="color:purple;font-family:Courier New;">tidyverse</span>
([Wickham et al. 2019](#ref-tidyverse)) y
<span style="color:purple;font-family:Courier New;">ggplot2</span>
([Wickham et al. 2025](#ref-R-ggplot2); [Wickham
2016](#ref-ggplot22016)).

## Procesado de los datos

Tras comprobar que no hay datos perdidos con `is.na()` y `sum()`,
procesamos el dataset para seleccionar únicamente las variables con las
que trabajaremos. Usamos las funciones `mutate()`, `relocate()` y
`select()`, además de `rename()` para cambiar el nombre a las columnas.
Para comprobarlo, elaboramos la siguiente tabla para visualizar las 10
primeras filas del dataset procesado.

<table class="table table-bordered">

<caption>

Datos procesados
</caption>

<thead>

<tr>

<th style="text-align:left;">

Fármaco
</th>

<th style="text-align:left;">

Ruta_metabólica
</th>

<th style="text-align:left;">

Diana_celular
</th>

<th style="text-align:right;">

AUC
</th>

<th style="text-align:left;">

Sensibilidad
</th>

<th style="text-align:right;">

LN_IC50
</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.930220
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-1.463887
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.614970
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-4.869455
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.791072
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-3.360586
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.592660
</td>

<td style="text-align:left;">

Media
</td>

<td style="text-align:right;">

-5.044940
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.734047
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-3.741991
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.582439
</td>

<td style="text-align:left;">

Media
</td>

<td style="text-align:right;">

-5.142961
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.867348
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-1.235034
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.834067
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-2.632632
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.821438
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-2.963191
</td>

</tr>

<tr>

<td style="text-align:left;">

Camptothecin
</td>

<td style="text-align:left;">

DNA replication
</td>

<td style="text-align:left;">

TOP1
</td>

<td style="text-align:right;">

0.905050
</td>

<td style="text-align:left;">

Baja
</td>

<td style="text-align:right;">

-1.449138
</td>

</tr>

</tbody>

</table>

## Variables escogidas

Para cumplir con los objetivos establecidos, se seleccionaron las
siguientes variables:

- **Fármaco**: Nombres de los fármacos anticancerígenos.

- **Ruta_metabólica**: Nombres de las rutas o vías de señalización.

- **Diana_celular**: Distintas dianas sobre las que actúan los fármacos.

- **AUC**: El AUC (área bajo la curva) mide la concentración plasmática
  del fármaco en el organismo durante el tiempo tras su administración.
  Representa el área que queda bajo la curva de concentración del
  fármaco frente al tiempo tras administrarlo, por lo que sus unidades
  combinan concentración y tiempo (por ejemplo, µg·h/mL). Así, se puede
  determinar qué cantidad de fármaco llega a las células.

- **LN_IC50**: El IC50 es una medida que representa la concentración de
  un fármaco que reduce la actividad de una enzima, célula o receptor en
  un 50% respecto a su actividad normal. Cuanto más bajo sea, más
  potente es el fármaco ya que se necesita menos concentración para
  alcanzar el mismo nivel de inhibición. Se expresa con su logaritmo
  para facilitar su análisis y representación gráfica.

- **Sensibilidad**: Se trata de una variable que hemos creado para
  encasillar los distintos valores de IC50 en niveles (alto, medio y
  bajo) dependiendo de si son:

  - \< -7 alto
  - -7 a -5 medio
  - \> -5 bajo

## Análisis estadísticos

### t de student

Este test se utiliza para comparar una variable dependiente con otra
independiente categórica con 2 grupos.

Utilizaremos este análisis para comparar los fármacos que actúan sobre
la de señalización JNK y p38, **Doramapimod** (inhibidor de p38) e
**inhibidor VIII de JNK**. Esta señalización se refiere a dos rutas de
MAPK que actúan principalente en respuestas a estrés celular e
inflamación para regular distintos procesos como la expresión génica,
apoptosis, proliferación y diferenciación. Se estudian en cáncer porque
están implicadas en el desarrollo de esta enfermedad.

Para realizar el análisis, compararemos los AUC de ambos fármacos para
comprobar si hay diferencias en la concentración de cada uno de estos
que llega a las células en un tiempo determinado. Es decir, estaremos
comparando la exposición de las células a cada uno de los fármacos desde
su administración.

Para llevar a cabo el análisis, primero seleccionamos solo las
observaciones de la señalización JNK y p38. Luego, aplicamos el test.

``` r
JNK_p38 <- DataProc %>% 
  filter(Ruta_metabólica == "JNK and p38 signaling") #Filtramos por la vía de señalización sobre la que haremos el análisis

t.test(AUC ~ Fármaco, data = JNK_p38)

#En este punto, mostramos el código y ocultamos los resultados, que se verán más adelante
```

Elaboramos un gráfico boxplot para visualizar los resultados de la
siguiente forma:

``` r
JNK_p38 %>% 
  ggplot(aes(x = Fármaco, y = AUC, colour = Fármaco)) + 
  geom_boxplot(outlier.shape = NA, width = 0.9) +
  geom_jitter(width = 0.18, alpha = 0.1, size = 0.75) + #Editamos los parámetros para que los puntos no se vean muy superpuestos
  coord_cartesian(ylim = c(0.86, 1.01)) + #Ajustamos el eje y para que se vean las cajas de forma más clara
  labs(
    title = "Exposición celular a inhibidores de la vía JNK y p38",
    x = "Fármaco",
    y = "Área bajo la curva (AUC)"
  ) +
  theme(
    legend.position = "none",
    plot.title = element_text(hjust = 0.5)
  )
```

### chi cuadrado

Este test se utiliza para evaluar la asociación entre dos variables
categóricas, permitiendo determinar si la distribución de una variable
depende de los niveles de la otra.

Utilizaremos este análisis para estudiar la relación entre las rutas
metabólicas y los niveles de sensibilidad a fármacos anticancerígenos,
clasificada en distintos niveles (alta, media y baja) en función de la
respuesta celular observada.

Para realizar el análisis, se comparará la frecuencia de cada nivel de
sensibilidad entre las distintas rutas metabólicas, con el objetivo de
comprobar si existen diferencias en la respuesta a los fármacos según la
ruta implicada. Es decir, se evaluará si determinadas rutas se asocian a
una mayor o menor sensibilidad a los tratamientos anticancerígenos.

Para llevar a cabo el análisis, primero se construye una tabla de
contingencia entre la ruta metabólica y el nivel de sensibilidad, sobre
la cual se aplica el test x<sup>2</sup> de independencia.

``` r
tabla_chi <- table(DataProc$Ruta_metabólica, DataProc$Sensibilidad)
tabla_chi #Creamos una tabla de contingencia a partir de la cual se realiza el análisis

chi <- chisq.test(tabla_chi)
chi
```

Elaboramos un gráfico de barras para visualizarlo.

``` r
DataProc %>% 
  ggplot(aes(x = Sensibilidad, fill = Ruta_metabólica)) + #Creamos barras para cada nivel de sensibilidad que se coloreen en función de las rutas metabólicas
  geom_bar(position = "fill") +
  labs(x = "Nivel de sensibilidad", y = "")
```

### ANOVA

El test ANOVA (Analysis of Variance) es una prueba estadística que se
utiliza para comparar las medias de más de dos grupos y determinar si
existen diferencias significativas entre ellos. En lugar de hacer
múltiples comparaciones individuales, ANOVA evalúa si la variabilidad
entre grupos es mayor que la variabilidad dentro de los grupos, lo que
indicaría un efecto real del factor estudiado.

En este caso, utilizamos ANOVA para comparar la sensibilidad a los
fármacos entre distintas dianas celulares, usando la medida del LN_IC50,
que refleja la sensibilidad al tratamiento. Cada diana celular
representa un grupo distinto, y con ANOVA evaluamos si la eficacia de
los tratamientos varía según la diana tratada

``` r
DataProc$Diana_celular <- as.factor(DataProc$Diana_celular)

anova <- aov(LN_IC50 ~ Diana_celular, data = DataProc)

summary(anova) #Con summary mostramos el resultado de p value
```

A continuación, elaboramos un gráfico para visualizarlo.Como utilizamos
un número demasiado grande de dianas celulares, para el gráfico solo
escogemos aquellas que están en los extremos de más y menos
sensibilidad.

``` r
stats <- DataProc %>%
  group_by(Diana_celular) %>%
  summarise(mediana = median(LN_IC50, na.rm = TRUE)) %>%
  arrange(mediana)

top <- bind_rows(
  head(stats, 10), #Seleccionamos solo las dianas que están en los extremos para que el gráfico sea legible y entendible
  tail(stats, 10)
)

ggplot(top, aes(x = reorder(Diana_celular, mediana), y = mediana)) +
  geom_col(fill = "pink") + #Cambiamos el color de las barras
  coord_flip() +
  labs(
    x = "Diana celular",
    y = "Mediana LN(IC50)",
    title = "Dianas más y menos sensibles"
  ) +
  theme_minimal()
```

# Resultados

## Análisis 1

***t de student y gráfico boxplot***

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  AUC by Fármaco
    ## t = -5.7018, df = 1502, p-value = 1.425e-08
    ## alternative hypothesis: true difference in means between group Doramapimod and group JNK Inhibitor VIII is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.009099937 -0.004441434
    ## sample estimates:
    ##        mean in group Doramapimod mean in group JNK Inhibitor VIII 
    ##                        0.9665484                        0.9733191

![](index_files/figure-gfm/t%20grafico%20result-1.png)<!-- -->

## Análisis 2

***x<sup>2</sup> y gráfico de barras***

    ##                                    
    ##                                      Alta  Baja Media
    ##   ABL signaling                         0   949     6
    ##   Apoptosis regulation                  1 10470   357
    ##   Cell cycle                            0 11572    48
    ##   Chromatin histone acetylation         5  7674   483
    ##   Chromatin histone methylation         0 10612     0
    ##   Chromatin other                       0  8576     0
    ##   Cytoskeleton                          0  3387     0
    ##   DNA replication                      13 17325   312
    ##   EGFR signaling                        0  6733     1
    ##   ERK MAPK signaling                    4 13308    38
    ##   Genome integrity                      0 12221     0
    ##   Hormone-related                       0  3820     0
    ##   IGF1R signaling                       0  5512     0
    ##   JNK and p38 signaling                 0  1905     0
    ##   Metabolism                           26  4623   153
    ##   Mitosis                              22  5396  1035
    ##   Other                                54 20871   477
    ##   Other, kinases                        8 17258    11
    ##   p53 pathway                           0  3776     0
    ##   PI3K/MTOR signaling                   8 22671    45
    ##   Protein stability and degradation     2  6626   459
    ##   RTK signaling                         0 10514    59
    ##   Unclassified                          0 24978     1
    ##   WNT signaling                         0  7631     0

    ## 
    ##  Pearson's Chi-squared test
    ## 
    ## data:  tabla_chi
    ## X-squared = 15133, df = 46, p-value < 2.2e-16

![](index_files/figure-gfm/gráfico%20chi%20result-1.png)<!-- -->

## Análisis 3

***ANOVA y gráfico de barras divergentes***

    ##                   Df  Sum Sq Mean Sq F value Pr(>F)    
    ## Diana_celular    183 1158617    6331    2394 <2e-16 ***
    ## Residuals     213980  565873       3                   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 27872 observations deleted due to missingness

![](index_files/figure-gfm/ANOVA%20result%20grafico-1.png)<!-- -->

# Discusión

## Efecto de dos fármacos distintos sobre la vía de señalización JNK y p38

El resultado del análisis t de student dió p \< 0,05; por lo que
rechazamos la hipótesis nula y podemos aceptar la alternativa. Las
medidas de sensibilidad entre los fármacos que actúan sobre la vía de
señalización JNK y p38 muestran diferencias significativas.

El gráfico boxplot proporciona un representación visual de esto, ya que
podemos obervar como el Doramapimod presenta valores de AUC más bajos
que el inhibidor VIII de JNK, por lo que el primero presentaría niveles
de sensibilidad celular superiores.

## Niveles de sensibilidad según la ruta metabólica

El resultado del análisis x<sup>2</sup> muestra p \< 0,05; por lo que se
rechaza la hipótesis nula y aceptamos la alternativa. Hay diferencias
significativas en los niveles de sensibilidad a fármacos
anticancerígenos entre las distintas rutas metabólicas.

El gráfico de barras muestra de forma visual clara cómo los patrones de
sensibilidad cambian según la ruta, de forma que algunas se relacionan
muy estrechamente con alta, media o baja sensibilidad.

## Respuesta a fármacos según la diana celular

El resultado del análisis ANOVA muestra p \< 0,05, por lo que se rechaza
la hipótesis nula y se acepta la alternativa. La respuesta a fármacos
muestra diferencias significativas entre las distintas dianas celulares.

El gráfico de barras divergentes muestra las dianas celulares más y
menos sensibles según la mediana de IC50, de forma que las barras
positivas muestran las dianas menos sensibles y las más negativas
representan aquellas con mayor sensibilidad.

# Conclusión

Como conclusión final, podemos afirmar que existen patrones de respuesta
a fármacos anticancerígenos según la sensibilidad frente a ellos que
presentan las células fuertemente asociados a distintos factores como lo
son el fármaco en sí, la ruta metabólica o vía de señalización en la que
interfieren y la diana celular a la que van dirigidos.

# Bibliografía

<div id="refs" class="references csl-bib-body hanging-indent"
entry-spacing="0">

<div id="ref-ggplot22016" class="csl-entry">

Wickham, Hadley. 2016. *Ggplot2: Elegant Graphics for Data Analysis*.
Springer-Verlag New York. <https://ggplot2.tidyverse.org>.

</div>

<div id="ref-tidyverse" class="csl-entry">

Wickham, Hadley, Mara Averick, Jennifer Bryan, Winston Chang, Lucy
D’Agostino McGowan, Romain François, Garrett Grolemund, et al. 2019.
*Tidyverse: Easily Install and Load the ’Tidyverse’*.
<https://CRAN.R-project.org/package=tidyverse>.

</div>

<div id="ref-R-readxl" class="csl-entry">

Wickham, Hadley, and Jennifer Bryan. 2023. *Readxl: Read Excel Files*.
<https://readxl.tidyverse.org>.

</div>

<div id="ref-R-ggplot2" class="csl-entry">

Wickham, Hadley, Winston Chang, Lionel Henry, Thomas Lin Pedersen,
Kohske Takahashi, Claus Wilke, Kara Woo, Hiroaki Yutani, Dewey
Dunnington, and Teun van den Brand. 2025. *Ggplot2: Create Elegant Data
Visualisations Using the Grammar of Graphics*.
<https://ggplot2.tidyverse.org>.

</div>

<div id="ref-R-dplyr" class="csl-entry">

Wickham, Hadley, Romain François, Lionel Henry, Kirill Müller, and Davis
Vaughan. 2023. *Dplyr: A Grammar of Data Manipulation*.
<https://dplyr.tidyverse.org>.

</div>

<div id="ref-knitr2014" class="csl-entry">

Xie, Yihui. 2014. “Knitr: A Comprehensive Tool for Reproducible Research
in R.” In *Implementing Reproducible Computational Research*, edited by
Victoria Stodden, Friedrich Leisch, and Roger D. Peng. Chapman;
Hall/CRC.

</div>

<div id="ref-knitr2015" class="csl-entry">

———. 2015. *Dynamic Documents with R and Knitr*. 2nd ed. Boca Raton,
Florida: Chapman; Hall/CRC. <https://yihui.org/knitr/>.

</div>

<div id="ref-R-knitr" class="csl-entry">

———. 2023. *Knitr: A General-Purpose Package for Dynamic Report
Generation in r*. <https://yihui.org/knitr/>.

</div>

<div id="ref-yang2012genomics" class="csl-entry">

Yang, Wanjuan, Jorge Soares, Patricia Greninger, Elena J Edelman, Howard
Lightfoot, Simon Forbes, Nidhi Bindal, et al. 2012. “Genomics of Drug
Sensitivity in Cancer (GDSC): A Resource for Therapeutic Biomarker
Discovery in Cancer Cells.” *Nucleic Acids Research* 41 (D1): D955–61.

</div>

</div>
