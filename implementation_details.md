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

Arquitectura del bloque residual utilizado. Una mini versión del bloque en [this description](http://predictioncenter.org/casp13/doc/presentations/Pred_CASP13-DeepLearning-AlphaFold-Senior.pdf)

The network has been trained with 134 proteins and evaluated with 16 more. Clearly unsufficient data, but memory constraints didn't allow for more. Comparably, AlphaFold was trained with 29k proteins.

The output of the network is, then, a classification among 6 classes wich are ranges of distances between a pair of AAs. Here there's an example of AlphaFold predicted distances and the distances predicted by our model:

<div style="text-align:center">
	<img src="imgs/alphafold_preds.png", width="600">
</div>
Ground truth (left) and predicted distances (right) by AlphaFold.

<div style="text-align:center">
	<img src="models/distance_pipeline/images/golden_img_v91_45.png", width="900">
</div>
Ground truth (left) and predicted distances (right) by MiniFold.

The architecture of the Residual Network for distance prediction is very simmilar, the main difference being that the model here described was trained with windows of 200x200 AAs while AlphaFold was trained with crops of 64x64 AAs. When it comes to prediction, AlphaFold used the smaller window size to average across different outputs and achieve a smoother result. Our prediction, however, is a unique window, so there's no average (noisier predictions).


### Angles prediction

The ResNet for angles prediction is built as a 1D-ResNet and takes as input tensors of shape LxN. The window length is set to 34 and we only train and predict aangles of proteins with less than 200 (L) AAs. No larger proteins nor crops of larger proteins are used.

The 42 (N) channels of the input are distributed as follows: 20 for AAs in one-hot encoding (Lx20), 2 for the Van der Waals radius and the surface accessibility of the AA encoded previously and 20 channels for the Position Specific Scoring Matrix).

We followed the ResNet20 architecture but replaced the 2D Convolutions by 1D convolutions. The network output consists of a vector of 4 numbers that represent the `sin` and `cos` of the 2 dihedral angles between two AAs (Phi and Psi).

Dihedral angles were extracted from raw coordinates of the protein backbone atoms (N-terminus, C-alpha and C-terminus of each AA). The plot of Phi and Psi recieves the name of Ramachandran plot: 

<div style="text-align:center">
	<img src="imgs/ramachandran_plot.png">
</div>
The cluster observed in the upper-left region corresponds to the angles comprised between AAs when they form a Beta-sheet while the cluster observed in the central-left region corresponds to the angles comprised between AAs when they form an Alpha-helix.

The results of the model when making predictions can be observed below:
<div style="text-align:center">
	<img src="imgs/angle_preds.png">
</div>

The network has been trained with crops 38,7k crops from 600 different proteins and evaluated with some 4,3k more.

The architecture of the Residual Network is different from the one implemented in AlphaFold. The model here implemented was inspired by [this paper](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005324) and [this one](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0205819).

## Results
While the architectures implemented in this first preliminary version of the project are inspired by papers with great results, the results here obtained are not as good as they could be. It's probable that the lack of Multiple Alignmnent (MSA), MSA-based features, Physicochemichal properties of AAs (beyond Van der Waals radius) or the lack of both model and feature engineering have affected the models negatively, as well as the little data that they have been trained on. 

For that reason, we can conclude that it has been a somehow naive approach and we expect to further implement some ideas/improvements to these models. As the DeepMind team says: *"With few or no alignments accuracy is much worse"*. It would be interesting to use the predictions made by the models as constraints to a folding algorithm (ie. Rosetta) in order to visualize our results.

## References
* [DeepMind original blog post](https://deepmind.com/blog/alphafold/)
* [AlphaFold @ CASP13: “What just happened?”](https://moalquraishi.wordpress.com/2018/12/09/alphafold-casp13-what-just-happened/#s2.2)
* [Accurate De Novo Prediction of Protein Contact Map by Ultra-Deep Learning Model](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005324)
* [AlphaFold slides](http://predictioncenter.org/casp13/doc/presentations/Pred_CASP13-DeepLearning-AlphaFold-Senior.pdf)
* [De novo protein structure prediction using ultra-fast molecular dynamics simulation](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0205819)
