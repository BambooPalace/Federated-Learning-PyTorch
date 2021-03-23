# Federated-Learning-with-Differential Privacy

This repo is built upon [AshwinRJ](https://github.com/AshwinRJ/Federated-Learning-PyTorch) 's  implementation of the [vanilla federated learning paper](https://arxiv.org/abs/1602.05629) on image classificaton task of MNIST dataset. 

As the scope of my project, I fixed code on running FL experiment with CIFAR10 (both IID and non-IID). In case of non-IID, the data amongst the users can be split equally or unequally.

As the main goal of this project and new content on the earlier repo, a more complex task of image segmentation was implementated with federated learning, with  option of IID and non-IID(equal and unequal data distribution), and option of training with differential privacy with Opacus library.

Many arguments are provided with this code for tuning all the necessary hyperparameters for training image segmentation with federated learning and differential privacy. Details are included in the `Options` section below.

## Requirments
Install all the packages from requirements.txt
* Python3
* Pytorch
* Torchvision
* Opacus
* Pycocotools

## Data
* The data for image classification: MNIST and CIFAR10 will be automatically downloaded into the `data` subfolder when runing the code in `classification' part.
* Both the image and annotation data for image segmentation has to be manual downloaded in the `data/folder` before runing experiments on segmentation. The data ready to use has to be in the format of coco dataset, i.e. torchvision.datasets.CocoDetection format. Please refer to the COCO [website](https://cocodataset.org/#download) on how to handle the data.

## Running the experiments
Code will automatically run on gpu if gpu and cuda is available on the machine.
## Classification

* To run the baseline (no federated learning) experiment with MNIST (or CIFAR) on MLP:
```
python classification/baseline_main.py --model=mlp --dataset=mnist --epochs=10
```
-----

* To run the federated experiment with CIFAR on CNN (IID):
```
python classification/federated_main.py --model=cnn --dataset=cifar --iid=1 --epochs=10
```
* To run the same experiment under non-IID equal data distribution condition:
```
python classification/federated_main.py --model=cnn --dataset=cifar --iid=0 --epochs=10
```
* To run the same experiment under non-IID unequal data distribution condition:
```
python classification/federated_main.py --model=cnn --dataset=cifar --iid=0 --unequal=1 --epochs=10
```

## Segmentation
* To run the baseline experiment with COCO (train2017 or val2017 dataset) on lraspp_mobilenetv3 model.
```
python segmentation/baseline_main.py --data=val2017 --model=lraspp_mobilenetv3 --num_classes=21 --epochs=10 --num_workers=4 --save_frequency=1  
```

* To run the federated learning experiments.
```
python segmentation/federated_main.py --model=lraspp_mobilenetv3  --num_classes=21 --activation=tanh --weight=0.5 --pretrained --iid=1 --unequal=0 --local_bs=8 --num_workers=4 --save_frequency=1 --data=val2017 --num_users=100 --local_test_frac=0.1 --local_ep=5  --epochs=5
```
* To run the federaated learning experiments with differential privacy.

(As Opacus is a developing library, the output generated many warnings during runtime, so I suggest ignore warning option as below in the command line.)
```
python -W ignore segmentation/federated_main.py --model=lraspp_mobilenetv3 --num_classes=21 --pretrained  --iid=1 --unequal=0 --local_bs=8 --virtual_bs=8 --num_workers=1 --save_frequency=1 --data=val2017 --num_users=50 --local_test_frac=0.1 --no_dropout --lr=0.01 --activation=tanh --weight=0.5 --noise_multiplier=0.2 --max_grad_norm=5 --dp  --local_ep=5  --epochs=5
```

## Options for Segmentation
The default values for various paramters parsed to the experiment are given in ```options.py```. Details are given some of those parameters:

```
optional arguments:
  -h, --help            show this help message and exit
  --epochs EPOCHS       number of rounds of training
  --num_users NUM_USERS
                        number of users: K
  --frac FRAC           the fraction of clients used for training: C
  --local_ep LOCAL_EP   the number of local epochs: E, default is 10
  --local_bs LOCAL_BS   local batch size: B, default=8, local gpu can only set 1
  --num_workers NUM_WORKERS
                        test colab gpu num_workers=1 is faster
  --lr LR               learning rate
  --momentum MOMENTUM   SGD momentum (default: 0.9)
  --model {fcn_mobilenetv2,deeplabv3_mobilenetv2,deeplabv3_mobilenetv3,lraspp_mobilenetv3,fcn_resnet50}
                        model name
  --num_classes NUM_CLASSES
                        number of classes max is 81, pretrained is 21
  --cpu_only            indicate to use cpu only
  --optimizer {sgd,adam}
                        type of optimizer
  -aux AUX_LR, --aux_lr AUX_LR
                        times of normal learning rate used for auxiliary classifier
  --lr_scheduler {lambda,step}
                        learning rate scheduler
  --checkpoint CHECKPOINT
                        full file name of the checkpoint
  --save_frequency SAVE_FREQUENCY
                        number of epochs to save checkpoint
  --test_frequency TEST_FREQUENCY
                        number of epochs to eval global test data
  --train_only
  --pretrained          only available for deeplab_mobilenetv3 and lraspp_mobilenetv3
  --activation {relu,tanh}
                        set activatition function in models as argument
  --dataset DATASET     name of dataset
  --iid IID             Default set to IID. Set to 0 for non-IID.
  --unequal UNEQUAL     whether to use unequal data splits for non-i.i.d setting (use 0 for equal splits)
  --verbose VERBOSE     verbose
  --seed SEED           random seed
  --root ROOT           home directory
  --data {val2017,train2017}
                        val has 5k images while train has 118k
  --sample_rate SAMPLE_RATE
                        fraction of large dataset to be used for training, can reduce training memory
  --local_test_frac LOCAL_TEST_FRAC
                        frac of num_users for local testing
  --freeze_backbone     choose to not train backbone
  --weight WEIGHT       the weight assigned to computing loss of background class
  --focus_class FOCUS_CLASS
                        the only class that affect loss function, other class weight set as 0 except background
  --dp
  --virtual_bs VIRTUAL_BS
                        the bs for noise addition, to save memory
  --max_grad_norm MAX_GRAD_NORM
                        max per-sam norm to clip
  --noise_multiplier NOISE_MULTIPLIER
                        suggested range in 0.1~2
  --no_dropout          set dropout prob as 0
  --filename FILENAME   image filename for inference.
```


## Further Readings
### Papers:
* [Communication-Efficient Learning of Deep Networks from Decentralized Data](https://arxiv.org/abs/1602.05629)
* [Deep Learning with Differential Privacy](https://arxiv.org/abs/1607.00133)
* [MAKING THE SHOE FIT: ARCHITECTURES, INITIALIZATIONS, AND TUNING FOR LEARNING WITH PRIVACY](https://openreview.net/forum?id=rJg851rYwH)
### Blog Posts:
* [Federated learning comics](https://federated.withgoogle.com/)
* [Opacus (a pytorch DP library) FAQ](https://opacus.ai/docs/faq)

