# Hướng dẫn Tải về, Giải nén và Chạy Dự án HEBO
```bash
git clone https://github.com/leduythuczen/BO_4_LS.git
cd HEBO/BOiLS
```
## Setup
Our experiments were performed on two machines with **Intel Xeon CPU E5-2699 v4@2.20GHz**, 64GB RAM, running
**Ubuntu 18.04.4 LTS** and equipped with one **NVIDIA Tesla
V100** GPU. All algorithms were implemented in **CONDA /MINICONDA ( Python 3.7 )** relying on `ABC v1.01`.
( implement on any hardware that is considered similar)

### Environment
- Install yosys
```shell script
sudo apt-get update -y
sudo apt-get install -y yosys
```

- Create Python 3.7 venv

```shell script
# Create virtualenv
python3.7 -m venv ./venv

# Activate venv
source venv/bin/activate

# Try installing requirements
# if getting issues with torch installation visit: https://pytorch.org/get-started/previous-versions/
conda install --yes --file requirements.txt
#----- Begin Graph-RL: if you need to run Graph-RL experiments, you need to install the following (you can skip this if BOiLS is only what you need): 
# follow instructions from: https://github.com/krzhu/abc_py
#-----  End Graph-RL -----
#There maybe other libraies that can not be installed due to a variety of reasons, so they will be auto resolved by CONDA
#Due to that fact, along the implement proccess there might be other libraries needed to be considered for installation

```

### Dataset
Dataset and results should be **stored in the same directory** `STORAGE_DIRECTORY`: run `python utils_save.py` and follow 
the instructions given in the `FileNotFoundError` message. Rerun `python utils_save.py` to check 
where the data should be saved (`DATA_PATH`) and where the results will be stored.

- download the circuits from [EPFL Combinatorial Benchmark Suite](https://github.com/lsils/benchmarks) and put them 
(only the "*.blif" are needed) in 
`DATA_PATH/benchmark_blif/` (not in a subfolder as the code will look for the circuits directly as 
`DATA_PATH/benchmark_blif/*.blif`).

### Setup sanity-check for fair comparison
If comparing with our reported results, run the following in your environment and make sure the output statistics are the same:
```shell script
DATA_PATH=... # change with your DATA_PATH
yosys-abc  -c "read $DATA_PATH/benchmark_blif/sqrt.blif; strash; balance; rewrite -z; if -K 6; print_stats;"
# Should output:
#  top                           : i/o =  128/   64  lat =    0  nd =  4005  edge =  19803  aig  = 29793  lev = 1023
```

---
## Run experiments

The code is organised in a modular way, providing coherent API for all optimisation methods. Third-party libraries used for the baseline implementations can be found in 
the [resources](./resources) directory, while the scripts to run the synthesis flow optimisation experiments are in the 
[core](./core) folder. The only exception to this organisation is for `DRiLLS` algorithm whise implementation is stored in [DRiLLS](./DRiLLS).


#### Run BOiLS
**BOiLS** can be run as shown below to find a sequence of logic synthesis primitives optimising the area / delay of a given circuit (e.g. `log2.blif` from EPFL benchmark). 

```shell script
python ./core/algos/bo/boils/main_multi_boils.py --designs_group_id log2 --n_parallel $n_parallel 1 \
                      --seq_length 20 --mapping fpga --action_space_id extended --ref_abc_seq resyn2 \
                      --n_total_evals 200 --n_initial 20 --device 0 --lut_inputs 4 --use_yosys 1  \
                      --standardise --ard --acq ei --kernel_type ssk \
                      --length_init_discrete_factor .666 --failtol 40 \
                      --objective area \
                      --seed 0"
```
Meaning of all the parameters are provided in the script: [./core/algos/bo/hebo/multi_hebo_exp.sh](core/algos/bo/hebo/multi_hebo_exp.sh). We created similar scripts for a wide set of optimisers, as detailed in the following section.


#### Setup to run COMBO
To run sequence optimisation using [**COMBO**](https://github.com/QUVA-Lab/COMBO) you need to download code of the 
official implementation, and to put it in the `./resources/` folder. 

```shell script
cd resources
wget https://github.com/QUVA-Lab/COMBO/archive/refs/heads/master.zip
unzip master.zip
mv COMBO-master/ COMBO
```

