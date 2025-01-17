# HW2VEC: A Graph Learning Tool for Automating Hardware Security
[![Build Status](https://travis-ci.com/louisccc/hw2vec.svg?branch=master)](https://travis-ci.com/github/louisccc/hw2vec) [![GitHub license](https://img.shields.io/github/license/Sujit-O/pykg2vec.svg)](https://github.com/Sujit-O/pykg2vec/blob/master/LICENSE) ![PyPI version](https://badge.fury.io/py/hw2vec.svg)

**HW2VEC** is an open-source graph learning tool for hardware security applications.
**HW2VEC** provides an automated pipeline to extract a graph representation (abstract syntax tree or data flow graph) from a hardware design in various abstraction levels (RTL or gate-level netlist).
Besides, **HW2VEC** includes graph learning functional components for users to apply graph learning approaches to these hardware designs in non-Euclidean form according to their problem settings.
In this readme, we demonstrate how to use **HW2VEC** and provide its use-cases for two hardware security applications: Hardware Trojan Detection and IP Piracy Detection.
We hope that **HW2VEC** can be helpful for researchers and practitioners in hardware security research community. In this repo, we integrate [Pyverilog](https://github.com/PyHDI/Pyverilog) as part of our graph extraction pipeline (**HW2GRAPH**) and [Pytorch-Geometric](https://github.com/rusty1s/pytorch_geometric) into our graph learning pipeline (**GRAPH2VEC**). The architecture of **HW2VEC** is shown as follows:
![](https://github.com/AICPS/hw2vec/blob/master/figures/archi.png?raw=true)

# To Get Started (Updated by NMSU PEARL LAB)
We recommend our users to use Linux system and [Anaconda](https://www.anaconda.com/) as the virtual environment. The environment requirements for hw2vec is as follows,
- python >= 3.6 
- torch == 1.6.0
- torch_geometric == 1.6.1
- pygraphviz

You can either install hw2vec from pypi or clone our repo. Here we provide one recommended command sequence, 
```sh
# run the following commands if you choose to install hw2vec from pip instead of installing from pypi or cloning our repo.
$ conda create --name hw2vec python=3.6
$ conda activate hw2vec

# Install pytorch and cuda toolkit (Make sure you have cuda 10.1 installed on your machine)
$ conda install pytorch==1.6.0 torchvision==0.7.0 cudatoolkit=10.1 -c pytorch

# Install scipy:
$ conda install -c anaconda scipy=1.4.1

# Make sure you have cuda 10.1 installed on your machine. Run the following commands to install pytorch_geometric 1.6.1:
$ python -m pip install torch-scatter --no-cache-dir --no-index --find-links https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ python -m pip install torch-sparse --no-cache-dir --no-index --find-links https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ python -m pip install torch-cluster --no-cache-dir --no-index --find-links https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ python -m pip install torch-spline-conv --no-cache-dir --no-index --find-links https://pytorch-geometric.com/whl/torch-1.6.0+cu101.html
$ python -m pip install torch-geometric --no-cache-dir


# Afterwards, install the remaining dependencies:
$ pip install pyverilog pydot pygraphviz matplotlib 

```

This set of commands assumes you have cuda 10.1 in your local and you are using Linux. Please refer to the installation guides of [torch](https://pytorch.org/) and [pytorch_geometric](https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html) if you have different cuda settings. If installing pygraphviz in windows, please refer to this [issue](https://github.com/pygraphviz/pygraphviz/issues/58) for more information.

# Use Cases Examples

## Use Case 1: Transforming a hardware design to a graph then to a graph embedding
In this use case, we demonstrate how to use HW2VEC to transform a hardware design into a graph and then into an embedding with a pre-trained model. In the sample script examples/use_case_1.py, HW2GRAPH first uses preprocessing and graph generation modules to convert the hardware design _p_ into the corresponding graph _g_. Then _g_ is fed to GRAPH2VEC with the uses of Data Processing to generate _X_ and _A_. _X_ and _A_ are processed through the pre-trained model with Graph Convolution layers, Graph Pooling layers, and Graph Readout operations to generate the graph embedding _h<sub>g</sub>_. This  resulting _h<sub>g</sub>_ can be further inspected with the utilities of the model.

To run this use case, unzip the assets/datasets.zip then use the following commands:
```sh
$ cd examples
$ python use_case_1.py
```
Users can refer to the following code piece in use_case_1.py to configure the hardware design code path and pre-trained model weights and config paths.
```python
hw_design_dir_path = Path("../assets/TJ-RTL-toy/TjFree/det_1011/") # Change this path to other hardware design folder path.
pretrained_model_weight_path = "../assets/pretrained_DFG_TJ_RTL/model.pth" # Change this path to your desired pretrained model weight path.
pretrained_model_cfg_path = "../assets/pretrained_DFG_TJ_RTL/model.cfg" # Change this path to your desired pretrained model config path.
cfg.graph_type = "DFG" # each pretrained model is bundled with one graph type so you will need to change this to fit the used pretrained model.
```
The expected embedding _h<sub>g</sub>_ is: 
```python
tensor([[1.5581e-02, 2.5182e-01, 2.1535e-02, 3.0264e-02, 3.3349e-02, 4.6067e-02,
         5.5791e-02, 3.6810e-02, 8.3800e-02, 5.1623e-02, 1.1715e-03, 1.2781e-01,
         ...      
         1.2203e-01, 1.2821e-01]], grad_fn=<CppNode<ScatterMax>>)
```

## Use Case 2: Hardware Trojan Detection
In this use case, we demonstrate how to use HW2VEC to detect hardware trojans (HT), which are intentional, malicious modifications of circuits by attackers. examples/use_case_2.py implements a proposed GNN-based approach to model the circuit's behavior and identify the presence of HTs. The dataset used in this use case is obtained from a famous trojan [benchmark](https://www.trust-hub.org/benchmarks/trojan). The converted hardware DFG dataset can be downloaded from [here](http://ieee-dataport.org/4618).

To realize the model with HW2VEC, we first use HW2GRAPH to convert each hardware design _p_ into a graph _g_. Then, we transform each _g_ to a graph embedding _h<sub>g</sub>_. Lastly, _h<sub>g</sub>_ is used to make a prediction with an MLP layer. To train the model, the cross-entropy loss _L_ is calculated collectively for all the graphs in the training set. 

To run this use case, use the script examples/use_case_2.py with the toy dataset (assets/TJ-RTL-toy). To train the model on the dataset, we provide the following command sequences.
```sh
$ cd examples
# for running HT detection on our toy RTL dataset using DFG graph type
$ python use_case_2.py --yaml_path ./example_gnn4tj.yaml --raw_dataset_path ../assets/TJ-RTL-toy --data_pkl_path dfg_tj_rtl.pkl --graph_type DFG (--device cuda)

# for running HT detection on our toy RTL dataset using AST graph type
$ python use_case_2.py --yaml_path ./example_gnn4tj.yaml --raw_dataset_path ../assets/TJ-RTL-toy --data_pkl_path ast_tj_rtl.pkl --graph_type AST (--device cuda)
```
Users can adjust the configuration (example_gnn4tj.yaml) to play with the model's hyperparameters.
```python
---
learning_rate: 0.001 # The initial learning rate for the model.
seed: 0 # Random seed.
epochs: 200 # Number of epochs to train.
hidden: 200 # Number of hidden units.
dropout: 0.5 # Dropout rate (1 - keep probability).
batch_size: 4 # Number of graphs in a batch.
num_layer: 2 # Number of layers in the neural network.
test_step: 10 # The interval between mini evaluation along the training process.
pooling_type: "topk" # Graph pooling type.
readout_type: "max" # Readout type.
ratio: 0.8 # Dataset splitting ratio
poolratio: 0.8 # Ratio for graph pooling.
embed_dim: 2 # The dimension of graph embeddings.
```

Users can refer to the following code piece in use_case_2.py for model configuration and modify it directly.
```python
convs = [
    GRAPH_CONV("gcn", data_proc.num_node_labels, cfg.hidden),
    GRAPH_CONV("gcn", cfg.hidden, cfg.hidden)
]
model.set_graph_conv(convs)

pool = GRAPH_POOL("sagpool", cfg.hidden, cfg.poolratio)
model.set_graph_pool(pool)

readout = GRAPH_READOUT("max")
model.set_graph_readout(readout)

output = nn.Linear(cfg.hidden, cfg.embed_dim)
model.set_output_layer(output)
```

Some of the performance metrics that we can provide are as follows:
| Graph Type |  Precision  |   Recall   |  F1 Score  |
|    :---:   |    :---:    |    :---:   |    :---:   |
|    DFG     |    0.8750   |   1.0000   |   0.9333   |
|    AST     |    0.8750   |   1.0000   |   0.9333   |


## Use Case 3: IP Piracy Detection
This use case demonstrates how to use HW2VEC to confront IP piracy - determining whether one of the two hardware designs is stolen from the other. The implemented method addresses IP piracy by assessing the similarities between hardware designs with a GNN-based model. The dataset used in this use case is obtained from a famous trojan [benchmark](https://www.trust-hub.org/benchmarks/trojan). The converted hardware DFG dataset can be downloaded from [here](http://ieee-dataport.org/4618). 

To implement the approach proposed, the GNN model has to be trained with a graph-pair classification trainer in GRAPH2VEC. The first step is to convert a pair of circuit designs _p<sub>1</sub>_, _p<sub>2</sub>_ to a pair of graphs _g<sub>1</sub>_, _g<sub>2</sub>_ with HW2GRAPH. Then, GRAPH2VEC transforms them into graph embeddings _h<sub>g1</sub>_, _h<sub>g2</sub>_. To assess the similarity of _h<sub>g1</sub>_ and _h<sub>g2</sub>_, a cosine similarity is computed as the final prediction of piracy. 

To run this use case, unzip the assets/datasets.zip and use the script examples/use_case_3.py with a toy dataset (assets/IP-RTL-toy or assets/IP-Netlist-toy). To train the model, we provide the following command sequences.
```sh
$ cd examples
# for running IP piracy detection on our toy RTL dataset using DFG graph type
$ python use_case_3.py --yaml_path ./example_gnn4ip.yaml --raw_dataset_path ../assets/IP-RTL-toy --data_pkl_path dfg_ip_rtl.pkl --graph_type DFG (--device cuda)
# for running IP piracy detection on our toy RTL dataset using AST graph type
$ python use_case_3.py --yaml_path ./example_gnn4ip.yaml --raw_dataset_path ../assets/IP-RTL-toy --data_pkl_path ast_ip_rtl.pkl --graph_type AST (--device cuda)
# for running IP piracy detection on our toy Netlist dataset using DFG graph type
$ python use_case_3.py --yaml_path ./example_gnn4ip.yaml --raw_dataset_path ../assets/IP-Netlist-toy --data_pkl_path dfg_ip_netlist.pkl --graph_type DFG (--device cuda)
# for running IP piracy detection on our toy Netlist dataset using AST graph type
$ python use_case_3.py --yaml_path ./example_gnn4ip.yaml --raw_dataset_path ../assets/IP-Netlist-toy --data_pkl_path ast_ip_netlist.pkl --graph_type AST (--device cuda)
```
Users can adjust the configuration (example_gnn4ip.yaml) to play with the model's hyperparameters.
```python
---
learning_rate: 0.001 # The initial learning rate for the model.
seed: 0 # Random seed.
epochs: 200 # Number of epochs to train.
hidden: 16 # Number of hidden units.
dropout: 0.5 # Dropout rate (1 - keep probability).
batch_size: 64 # Number of graphs in a batch.
num_layer: 5 # Number of layers in the neural network.
test_step: 10 # The interval between mini evaluation along the training process.
pooling_type: "sagpool" # Graph pooling type.
readout_type: "max" # Readout type.
ratio: 0.8 # Dataset splitting ratio
poolratio: 0.5 # Ratio for graph pooling.
embed_dim: 16 # The dimension of graph embeddings.
```

Users can refer to the following code piece for model configuration in use_case_3.py and modify it directly.
```python
convs = [
    GRAPH_CONV("gcn", data_proc.num_node_labels, cfg.hidden),
    GRAPH_CONV("gcn", cfg.hidden, cfg.hidden)
]
model.set_graph_conv(convs)

pool = GRAPH_POOL("sagpool", cfg.hidden, cfg.poolratio)
model.set_graph_pool(pool)

readout = GRAPH_READOUT("max")
model.set_graph_readout(readout)

output = nn.Linear(cfg.hidden, cfg.embed_dim)
model.set_output_layer(output)
```

Some of the performance metrics that we can provide are as follows:
| Graph Type |  Dataset  |  Accuracy  |  F1 Score  |
|    :---:   |    :---:  |   :---:    |    :---:   |
|    DFG     |    RTL    |   0.9841   |   0.9783   |
|    AST     |    RTL    |   0.8557   |   0.8333   |

# Citation
If you find our tool is useful in your research, please kindly consider citing our papers.
```
@misc{yu2021hw2vec,
      title={HW2VEC: A Graph Learning Tool for Automating Hardware Security}, 
      author={Shih-Yuan Yu and Rozhin Yasaei and Qingrong Zhou and Tommy Nguyen and Mohammad Abdullah Al Faruque},
      year={2021},
      eprint={2107.12328},
      archivePrefix={arXiv},
      primaryClass={cs.CR}
}
@TECHREPORT{UCI-TR-21-02,
  AUTHOR =        {Yasamin Moghaddas and Tommy Nguyen, Shih-Yuan Yu, Rozhin Yasaei, Mohammad Abdullah Al Faruque, “Technical Report for HW2VEC – A Graph Learning Tool for Automating Hardware Security“, CECS TR 21-02, posted on July 27, 2021},
  TITLE =         {Technical Report for HW2VEC – A Graph Learning Tool for Automating Hardware Security},
  NUMBER =        {TR-21-02},
  INSTITUTION =   {Center for Embedded and Cyber-Physical Systems University of California, Irvine},
  ADDRESS =       {Irvine, CA 92697-2620, USA},
  MONTH =         {July},
  YEAR  =         {2021},
   URL   =        {http://cecs.uci.edu/files/2021/07/TR-21-02.pdf}
}
```
