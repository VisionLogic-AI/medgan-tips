# A few additional basic tips on how to run Edward Choi's medGAN

Here is the link to Edward Choi's medGAN repository on GitHub: [link](https://github.com/mp2893/medgan). Congrats to his excellent work.

In this markdown, I add a few very basic details that complete Choi's `README.md` and can help run medGAN. My specs: Windows 10. I would like to thank [@ZwAnto](https://github.com/ZwAnto) for his assistance. The goal of this markdown is just to run medGAN and not to obtain useful results: we try to minimize the computing time at the cost of having poorly realistic generated samples.

Edward Choi's `medgan` repository is composed of two programs that have since been updated for Python 3:
* `process_mimic.py` (124 lines) inputs the public MIMIC-III dataset and outputs a suitable training dataset for `medgan.py`,
* `medgan.py` (410 lines) inputs the output of `process_mimic.py` and outputs the generated (fake) multi-label discrete patient records.

## 1) Process the MIMIC-III dataset with `process_mimic.py`.

First of all, we need to open _Anaconda Navigator_, then go to _Environments_, click on the right triangle next to _base (root)_ and _Open Terminal_: this opens a command prompt with the following path: `(base) C:\Users\<myusername>`.

In the command prompt, we change the directory to the folder where `ADMISSIONS.csv` and `DIAGNOSES_ICD.csv` from the MICMIC-III dataset (we only need these two) and the python codes are saved:
```
cd C:\Users\<myusername>\Documents\medgan-master
```
Still in the command prompt, we can then process the MIMIC-III dataset:
```
python process_mimic.py ADMISSIONS.csv DIAGNOSES_ICD.csv training-data "binary"
```
This will create 3 files in our folder: `training-data.matrix`, `training-data.pids` and `training-data.types`.

## 2) Run `medgan.py` using the `training-data.matrix` file generated by `process_mimic.py`.

With the command `python medgan.py --help`, we can see all the parameters we can choose.

Please read the NumPy version issue I added on Edward Choi's medGAN repository: [link](https://github.com/mp2893/medgan/issues/14).

Still with the command prompt, we create a `generated` folder in our `medgan-master` folder:
```
mkdir generated
```
Then we use the command:
```
python medgan.py training-data.matrix ./generated/samples --data_type="binary" --n_epoch=10 --n_pretrain_epoch=10
```
Once again, the goal here is just to run medGAN, not to obtain useful results: we try to minimize the computing time by taking small values for `n_epoch` and `n_pretrain_epoch`.
This will create 32 files in our `generated` folder: `checkpoint`, `samples`, `samples-0.data-00000-of-00001`, `samples-0.index`, `samples-0.meta`, `samples-1.data-00000-of-00001`,  etc. In `samples.txt`, we can check the values of `d_loss`, `g_loss`, `accuracy` and `AUC` at each epoch.
For comparison, the default value of `n_epoch` is 1000 and the default value of `n_pretrain_epoch` is 100.

## 3) Generate synthetic records.

We use the command:
```
python medgan.py training-data.matrix gen-samples --model_file=./generated/samples-9 --generate_data=True
 ```
Instead of 9, we take the last epoch (here we took 10 epochs starting from the index 0). This will create the `gen-samples.npy` file in the `medgan-master` folder.

In order to obain a csv file, we can execute the following script in Python:
```python
import numpy as np
import os
os.getcwd()
os.chdir('C:\\Users\\SYCB_CB\\Documents\\medgan-master')
data = np.load('gen-samples.npy')

import csv
with open('gen-samples.csv', 'w') as csvFile:
    writer = csv.writer(csvFile)
    writer.writerows(data)
csvFile.close()
 ```
This will create a `gen-samples.csv` file in the `medgan-master` folder.
