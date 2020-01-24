# MiniFold



## Reproducing the results

Estos son los siguientes pasos para ejecutar el código localmente o en la nube:

Nota: Como buena práctica, crea un ambiente virtual para poder reproducir el código.

### Pasos a seguir:

1. Clonar el repositorio: `git clone https://github.com/Tole8Tole/MiniFold.git`
2. Instalar dependencias: `pip install -r requirements.txt`
3. Obtener y dar formato a los datos:
	1. Descargar los datos [aquí](https://github.com/aqlaboratory/proteinnet) (seleccionar CASP7 text-based format)
	2. Descomprimir el archivo
	3. Recomendable crear un folder `/data` dentro del directorio `MiniFold` y copiar los archivos `training_30, training_70 and training90` a éste. Cambiar las extensiones a `.txt`.
4. Ejecutar los archivos tipo notebook (`preprocessing`) en el siguiente orden:
	4. `get_proteins_under_200aa.jl *source_path* *destin_path*`:  - selecciona proteínas de menos de 200 residuos de *source_path* file - (necesitas instalar [Julia programming language](https://julialang.org/) v1.0 para poder ejectutarlo)
	5. Correo el notebook `get_angles_from_coords_py.ipynb` - el cual realiza el cálculo de ángulos 'dihedrales' desde las coordenadas
	6. Correo el notebook `angle_data_preparation_py.ipynb`
5. Probar los modelos:
	7. Para **predicción de ángulos**: `models/predicting_angles.ipynb`
	8. Para **predicción de distancias**:
		1. `models/distance_pipeline/pretrain_model_pssm_l_x_l.ipynb`
		2. `models/distance_pipeline/pipeline_caller.py`

Si encuentras algunos detalles durante la ejecución puedes escribir a tole@ciencias.unam.mx



## Referencias
* [DeepMind original blog post](https://deepmind.com/blog/alphafold/)
* [AlphaFold @ CASP13: “What just happened?”](https://moalquraishi.wordpress.com/2018/12/09/alphafold-casp13-what-just-happened/#s2.2)
* [Siraj Raval's YT video on AlphaFold](https://www.youtube.com/watch?v=cw6_OP5An8s)
* [ProteinNet dataset](https://github.com/aqlaboratory/proteinnet)


Éste repositorio es un fork de:

`git clone https://github.com/EricAlcaide/MiniFold`

... donde probaremos otro tipo de estructuras.
 
Puedes clonar o darle "fork" a éste repositorio y poder contribuir en eĺ [código de conducta](https://thoughtbot.com/open-source-code-of-conduct)
 
## Meta

Autor original del repositorio:
 
* **Author's GitHub Profile**: [Eric Alcaide](https://github.com/EricAlcaide/)
* **Twitter**: [@eric_alcaide](https://twitter.com/eric_alcaide)
* **Email**: ericalcaide1@gmail.com