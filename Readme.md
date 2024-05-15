# RadarGNN

This repository contains an implementation of a graph neural network for the segmentation and object detection in radar point clouds. As shown in the figure below, the model architecture consists of three major components: Graph constructor, GNN, and Post-Processor.

![Model](https://github.com/Krutika-Mathapati/RadarGNN/assets/84244175/e934ea3f-42b4-49e4-adc7-ae9057e81a89)

The focus is on a modular implementation with the possibility to incorporate different invariances and equivariances into the overall model. It is built upon pytorch-geometric and provides usage with the [nuScenes](https://drive.google.com/file/d/1m0sWHNF-g-0mwy9KKudnuyaDRwMd1rVY/view?usp=sharing) dataset.

<br>

## Prerequisites
- OS: `Ubuntu 20.04 LTS`
- CUDA: `11.3`
- cuDNN: `8`
- Docker: `20.10`
- NVIDIA Container Toolkit
<br>

##  Preparation
To get started, first a project folder must be created. This folder contains everything related to this project. Inside the project folder create a "data" folder and within this folder, create a "results" subfolder. The trained models and evaluations will be stored in that folder. Depending on the desired dataset, create the following additional sub folders inside the "data" folder:
- datasets/nuscenes

In a second step follow the instructions of the [nuScenes](https://drive.google.com/file/d/1m0sWHNF-g-0mwy9KKudnuyaDRwMd1rVY/view?usp=sharing) websites to download and store the datasets in the created sub folders. 

Finally, clone this repository into the project folder using the command:
```
git clone https://github.com/TUMFTM/RadarGNN.git
```

<details>
<summary>If you use the nuScenes dataset, your folder structure should now look like this: </summary>

```
.
|  
+---data/  
|   |  
|   +---datasets/  
|   |   |  
|   |   +---nuScenes/  
|   |   |   |               
|   |   |   +---License.md
|   |   |   |
|   |   |   +---data/        
|   |
|   +---results/  
| 
+---docs/
|
+---configurations/
|   
+---test/
|
+---src/  
|
+---...
```
</details>
<br>

## Installation
It is recommended to use the provided [Dockerfile](Dockerfile) to build the development environment. However, if you don't want to use docker, you can also install the packages defined in [Dockerfile](Dockerfile) and [requirements.txt](requirements.txt) manually. To set up the environment with docker, run the following commands inside the repository folder:

1. Ensure the you are inside the repository folder.
```
cd RadarGNN/
```
2. Create the docker image.
```
docker build -t gnn:1.0 .
```
3. Run the container with GPU support and mount the "data" and "configurations" folder. By using the "-it" flag, the container is run in an interactive mode, so that you are directly inside the container after executing the command. Since the "data" and "configurations" folder are mounted, any changes in these folders are automatically mirrored from your local machine into the running docker container and vice versa. 
```
docker run -it --rm --gpus all -v ${local_path_to_data_folder}$:/app/data -v ${local_path_to_configurations_folder}$:/app/configurations gnn:1.0
```
4. Install the gnnradarobjectdetection package inside the container.
```
python3 -m pip install -e .
```
<br />

##  Usage
The overall pipeline is divided into three major steps. 

- Creation of a graph-dataset from the raw RadarScenes or nuScenes dataset
- Creation and training of a model based on the created graph-dataset
- Evaluation of the trained model

The settings of all three steps are defined in a unified configuration file, which must consequently be created first.
### 1. Create a configuration file 
The configuration file contains three sections with relevant settings for the corresponding steps (dataset creation, training, evaluation). It can be created based on the provided [configuration description](/configurations/configuration_description.yml) and [configuration template](/configurations/configuration_template.yml).
<br />

### 2. Create a graph-dataset
Next, the graph-dataset needs to be created by converting the radar point clouds of the raw datasets to a graph data structure. To do this, execute the following command inside the docker container: 
```
python3 src/gnnradarobjectdetection/create_dataset.py --dataset ${path_to_raw_dataset_folder}$ --config ${path_to_config_file}$
```

```
usage:          create_dataset.py [--dataset] [--config]

arguments:
    --dataset   Path to the raw (RadarScenes/nuScenes) dataset
    --config    Path to the created configuration.yml file
```

The created graph-dataset is saved in the automatically created folder "{path_to_dataset}/processed". After creating the graph-dataset, this folder may be renamed.
<br />

### 3. Create and train a model
In a next step, you can use the created graph-dataset to train a model. To do this, run the following command inside the docker container: 
```
python3 src/gnnradarobjectdetection/train.py --data ${path_to_graph_dataset_folder}$ --results ${path_to_results_folder}$ --config ${path_to_config_file}$
```
```
usage:             train.py [--data] [--results] [--config]

arguments:
    --data         Path to the created graph-dataset
    --results      Path to the created "results" folder
    --config       Path to the created configuration.yml file
```

Within the provided "results" folder, a new "model" folder is automatically created, in which the trained model is saved.
<br />

### 4. Evaluate a trained model 
Finally, you can evaluate a trained model using the following command inside the docker container: 
```
python3 src/gnnradarobjectdetection/evaluate.py --data ${path_to_graph_dataset_folder}$ --model ${path_to_model_folder}$ --config ${path_to_config_file}$ 
```
```
usage:             evaluate.py [--data] [--model] [--config]

arguments:
    --data         Path to the created graph-dataset (The same as used for the training of the model to evaluate)
    --model        Path to the folder in which the trained model is saved
    --config       Path to the created configuration.yml file
```
Within the provided "model" folder a new "evaluation" folder is created, in which the evaluation results are saved.

## Citation
If RadarGNN is useful or relevant to your research, please kindly recognize our contributions by citing our paper:

```bibtex
@article{fent2023radargnn,
  title={RadarGNN: Transformation Invariant Graph Neural Network for Radar-based Perception},
  author={Fent, Felix and Bauerschmidt, Philipp and Lienkamp, Markus},
  journal={arXiv preprint arXiv:2304.06547},
  year={2023}
}
```
