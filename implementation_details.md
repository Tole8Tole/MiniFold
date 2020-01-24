# Detalles de implementación

## Arquitectura propuesta

Los métodos implementados están inspirados en la publicación original de DeepMind. Se utilizan dos redes neuronales residuales diferentes (ResNets) para predecir ** ángulos ** entre aminoácidos adyacentes (AA) y ** distancia ** entre cada par de AA de una proteína. Para la predicción de distancia se usó un Resnet 2D mientras que para la predicción de ángulos se utilizó un Resnet 1D.

<div style="text-align:center">
	<img src="https://storage.googleapis.com/deepmind-live-cms/images/Origami-CASP-181127-r01_fig4-method.width-980.png" width="600" height="400">
</div>

Image from DeepMind's original blogpost.

### Distance prediction

### Predicción de distancia

La ResNet para la predicción de distancia se construye como una ResNet 2D y toma como tensores de entrada de forma LxLxN (una imagen normal sería LxLx3). La longitud de la ventana se establece en 200 (solo entrenamos y predecimos proteínas de menos de 200 AA) y las proteínas más pequeñas se rellenan para coincidir con el tamaño de la ventana. No se utilizan proteínas más grandes ni cultivos de proteínas más grandes.

Los 41 canales de la entrada se distribuyen de la siguiente manera: 20 para AA en codificación de un solo calor (LxLx20), 1 para el radio Van der Waals del AA codificado previamente y 20 canales para la Matriz de puntuación específica de posición).

La red se compone de paquetes de bloques residuales con la arquitectura a continuación ilustrada con bloques que recorren 1,2,4 y 8 zancadas más una primera capa convolucional normal y la última capa convolucional donde se aplica una función de activación Softmax para obtener una salida de LxLx7 (6 clases para diferentes distancias + 1 clase de basura para el relleno que está menos penalizado).

<div style="text-align:center">
	<img src="imgs/elu_resnet_2d.png">
</div>

Arquitectura del bloque residual utilizado. Una mini versión del bloque en [ésta descripción](http://predictioncenter.org/casp13/doc/presentations/Pred_CASP13-DeepLearning-AlphaFold-Senior.pdf)

La red ha sido entrenada con 134 proteínas y evaluada con 16 más. Datos claramente insuficientes, pero las restricciones de memoria no permitieron más. De manera comparable, AlphaFold fue entrenado con proteínas 29k.

La salida de la red es, entonces, una clasificación entre 6 clases que son rangos de distancias entre un par de AA. Aquí hay un ejemplo de las distancias predichas de AlphaFold y las distancias predichas por nuestro modelo:

<div style="text-align:center">
	<img src="imgs/alphafold_preds.png", width="600">
</div>
Verdad fundamental (izquierda) y distancias predichas (derecha) por AlphaFold.

<div style="text-align:center">
	<img src="models/distance_pipeline/images/golden_img_v91_45.png", width="900">
</div>
Verdad fundamental (izquierda) y distancias predichas (derecha) por MiniFold.

La arquitectura de la Red Residual para la predicción de distancia es muy similar, la principal diferencia es que el modelo aquí descrito fue entrenado con ventanas de 200x200 AA mientras que AlphaFold fue entrenado con cultivos de 64x64 AA. En lo que respecta a la predicción, AlphaFold utilizó el tamaño de ventana más pequeño para promediar las diferentes salidas y lograr un resultado más uniforme. Sin embargo, nuestra predicción es una ventana única, por lo que no hay un promedio (predicciones más ruidosas).



### Predicción de ángulos

La predicción de ResNet para ángulos está construida como 1D-ResNet y toma como tensores de entrada de forma LxN. La longitud de la ventana se establece en 34 y solo entrenamos y predecimos ángulos de proteínas con menos de 200 (L) AA. No se utilizan proteínas más grandes ni cultivos de proteínas más grandes.

Los 42 canales (N) de la entrada se distribuyen de la siguiente manera: 20 para AA en codificación de un solo calor (Lx20), 2 para el radio de Van der Waals y la accesibilidad a la superficie del AA codificado previamente y 20 canales para la puntuación específica de posición Matriz).

Seguimos la arquitectura ResNet20 pero reemplazamos las convoluciones 2D por convoluciones 1D. La salida de la red consiste en un vector de 4 números que representan el 'sin' y el 'cos' de los 2 ángulos diédricos entre dos AA (Phi y Psi).

Los ángulos diédricos se extrajeron de las coordenadas en bruto de los átomos del esqueleto de la proteína (N-terminal, C-alfa y C-terminal de cada AA). La trama de Phi y Psi recibe el nombre de gráfico de Ramachandran:

<div style="text-align:center">
	<img src="imgs/ramachandran_plot.png">
</div>
El grupo observado en la región superior izquierda corresponde a los ángulos comprendidos entre los AA cuando forman una hoja Beta, mientras que el grupo observado en la región central izquierda corresponde a los ángulos comprendidos entre los AA cuando forman una hélice alfa.

Los resultados del modelo al hacer predicciones se pueden observar a continuación:
<div style="text-align:center">
	<img src="imgs/angle_preds.png">
</div>

La red ha sido entrenada con cultivos de 38,7k cultivos de 600 proteínas diferentes y evaluada con unos 4,3k más.

La arquitectura de la red residual es diferente de la implementada en AlphaFold. El modelo implementado aquí fue inspirado por [éste paper](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005324) y [éste otro paper](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0205819).

## Resultados
Si bien las arquitecturas implementadas en esta primera versión preliminar del proyecto están inspiradas en documentos con excelentes resultados, los resultados aquí obtenidos no son tan buenos como podrían ser. Es probable que la falta de alineación múltiple (MSA), las características basadas en MSA, las propiedades fisicoquímicas de los AA (más allá del radio de Van der Waals) o la falta de ingeniería de modelos y características hayan afectado negativamente a los modelos, así como a los pocos datos que han sido entrenados en el mismo.

Por esa razón, podemos concluir que ha sido un enfoque de alguna manera ingenuo y esperamos implementar algunas ideas / mejoras en estos modelos. Como dice el equipo de DeepMind: * "Con pocas o ninguna alineación, la precisión es mucho peor" *. Sería interesante utilizar las predicciones hechas por los modelos como restricciones para un algoritmo de plegado (es decir, Rosetta) para visualizar nuestros resultados.


## Referencias
* [DeepMind original blog post](https://deepmind.com/blog/alphafold/)
* [AlphaFold @ CASP13: “What just happened?”](https://moalquraishi.wordpress.com/2018/12/09/alphafold-casp13-what-just-happened/#s2.2)
* [Accurate De Novo Prediction of Protein Contact Map by Ultra-Deep Learning Model](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005324)
* [AlphaFold slides](http://predictioncenter.org/casp13/doc/presentations/Pred_CASP13-DeepLearning-AlphaFold-Senior.pdf)
* [De novo protein structure prediction using ultra-fast molecular dynamics simulation](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0205819)
