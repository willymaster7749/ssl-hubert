## FedML Benchmark

### Examples

The example in this folder was taken from [FedML](https://github.com/FedML-AI/FedML/tree/master/python/examples/simulation/mpi_fedavg_datasets_and_models_example) repository on its release 0.7.300, using the configuration suggested on their
[benchmarking results](https://doc.fedml.ai/simulation/benchmark/BENCHMARK_MPI.html) for MPI-Based Federated Learning (fastest on this version).

### Data

FLUTE will automatically download the data used for this example, otherwise you can use the scripts provided [here](https://github.com/FedML-AI/FedML/tree/master/python/fedml/data) for each independent dataset in the FedML GitHub repository. 

### Run

If you downloaded the data manually, make sure that the variable `data_cache_dir` has been updated inside `preprocess.py`. Later, you can run the experiment as follows:

```code

    python -m torch.distributed.run  --nproc_per_node=4  e2e_trainer.py -dataPath ~/data -outputPath ~/outputTest  -config ./experiments/cv_lr_mnist/config.yaml -task cv_lr_mnist -backend nccl
```

### FedML Results

This comparison was carried out using Parrot (Simulator) on version 0.7.303 at commit ID [8f7f261f](https://github.com/FedML-AI/FedML/tree/8f7f261f44e58d0cb5a416b0d6fa270b42a91049). 

```
 _____________________________________________________________________________
|                    |   FedML (MPI) - Fastest   |   FLUTE (NCCL)  - Fastest  |
| Task               | Acc | Time     | GPU Mem  | Acc | Time     | GPU Mem   |
|--------------------|-----|----------|----------|-----|----------|-----------|
| LR_MNIST           | ~81 | 00:03:09 | ~3060 MB | ~81 | 00:01:35 | ~1060 MB  |
| CNN_FEMNIST        | ~83 | 05:49:52 | ~5180 MB | ~83 | 00:08:22 | ~1770 MB  |
| RESNET_FEDCIFAR100 | ~34 | 15:55:36 | ~5530 MB | ~33 | 01:42:01 | ~1900 MB  |
| RNN_FEDSHAKESPEARE | ~57 | 06:46:21 | ~3690 MB | ~57 | 00:21:50 | ~1270 MB  |
 -----------------------------------------------------------------------------
```

### FedML Configuration file

In order to reproduce this experiment in FedML please use the setup below. 

```yaml

common_args:
  training_type: "simulation"
  random_seed: 0

data_args:
  dataset: "mnist"
  data_cache_dir: ~/fedml_data
  partition_method: "hetero"
  partition_alpha: 0.5

model_args:
  model: "lr"

train_args:
  federated_optimizer: "FedAvg"
  client_id_list: "[]"
  client_num_in_total: 1000
  client_num_per_round: 10
  comm_round: 100
  epochs: 1
  batch_size: 10
  client_optimizer: sgd
  learning_rate: 0.03
  weight_decay: 0.001

validation_args:
  frequency_of_the_test: 20

device_args:
  worker_num: 10
  using_gpu: true
  gpu_mapping_file: config/fedemnist_cnn/gpu_mapping.yaml
  gpu_mapping_key: mapping_default # [3, 3, 3, 2]

comm_args:
  backend: "MPI"
  is_mobile: 0

```

### Flower Results

This comparison was carried out using Flower (Simulator) on version 1.0.0 at commit ID [4e7fad9](https://github.com/adap/flower/tree/4e7fad99389a5ee511730841b61f279e3359cb16). Showing that in some cases FLUTE can outperform 53x faster.

```
 ________________________________________________
|        |    Flower (Ray)   | FLUTE (NCCL/Gloo) |
|        | Acc |    Time     | Acc |    Time     |
|--------|-----|-------------|-----|-------------|
| CPU    | ~80 |   00:30:14  | ~80 |   00:03:20  |
| GPU 2x | ~80 |   01:21:44  | ~80 |   00:01:31  |
| GPU 4x | ~79 |   00:56:45  | ~81 |   00:01:26  |
 ------------------------------------------------
```

### Flower Configuration file

In order to reproduce this experiment in Flower please use the following patch [file](https://github.com/AnonymousQTHM31/FLUTE/blob/main/flower.patch) for the CPU setup. If you want to use multiple GPUs, follow the configuration suggested [here](https://github.com/adap/flower/issues/1415)