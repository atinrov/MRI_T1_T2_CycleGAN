
# CycleGAN Package
This project contains the implementation of a Cycle Generative Adversarial Network (CycleGAN) for various tasks, as package.


## Project Structure
```
├── cycle
│   ├── CallBacks.py
│   ├── CycleGAN.py
│   ├── DataMod.py
│   ├── Get_Process_Data.py
│   ├── HyperOptim.py
│   ├── Nets.py
│   ├── SetSeed.py
│   └── Set_FolderStructure.py
│   └──generation.py
│   └──ModelEvaluation.py

```

## File Descriptions

- `cycle/CallBacks.py`: CallBacks.py: This file contains two classes, CustomModelCheckpoint and CreateTensor, which extend the functionality of PyTorch Lightning's pl.Callback.

  * CustomModelCheckpoint: This class is used to save the state of a model during training. It can save the best model based on a specific metric and can also save model checkpoints periodically during training. In the on_validation_epoch_end method, it saves the best model checkpoint based on the monitored metric and in the on_train_epoch_end method, it saves a model checkpoint every N epochs.
  * CreateTensor: This class is used to save a PyTorch tensor representation of an image at the end of the first step in each validation epoch. The on_validation_batch_end method is responsible for this, saving the generated image tensor from the validation step outputs.
![UML Diagram from callbacks](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/CallBacks.svg)



- `cycle/CycleGAN.py`: It implements the CycleGAN model, a type of Generative Adversarial Network (GAN) that can transform images from one domain to another without paired data. It's a subclass of pl.LightningModule that encapsulates the CycleGAN model. It has several methods that define the training, validation, and test steps for the CycleGAN model.

   * Initialization: In the __init__ method, the model's hyperparameters are saved, and the automatic optimization flag is set to False to allow for more fine-grained control over the training process. The Generators (G_T1_T2 and G_T2_T1) and Discriminators (D_T1 and D_T2) are defined and initialized. The loss functions (identity loss, adversarial loss, and cycle loss) are also set up.

   * Forward Method :The forward method just passes the input through the G_T1_T2 generator.

   * training_step method:  the model's parameters are updated through several steps:

    1. The Discriminators' parameters are updated.
    2. The Generators' parameters are updated.
    3. The losses are logged.
    4. If the current epoch is greater than n_epochs, the learning rate schedulers are updated.

   * validation_step method: the Discriminators' and Generators' losses are computed without updating the model's parameters. The training metrics are computed, logged, and returned. If the current batch index matches log_every_n_iterations, the method also logs an image grid of the real and generated images.

   * test_step: This method works similarly to the validation_step method, but it operates on the test dataset.

   * configure_optimizers: This method defines the Adam optimizers for the Generators and Discriminators, along with learning rate schedulers. The method supports two modes: "linear" and "plateau". The "linear" mode uses a LambdaLR scheduler with a linear decay, while the "plateau" mode uses a ReduceLROnPlateau scheduler that lowers the learning rate when a metric has stopped improving. 
![UML Diagram from CycleGAN](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/CycleGAN.svg)

- `cycle/Get_Process_Data.py`: creates a pipeline for processing brain MRI images for training the deep learning model. The process involves multiple steps.

  1. Download: Fetching data from a specified URL, unzipping the .gz files, and storing them locally in NIfTI format.
  2. Brain Extraction: Applying HD-BET [(High-resolution Data-driven Brain Extraction Tool)](https://github.com/MIC-DKFZ/HD-BET)
  to remove non-brain tissue from the images.
  3. Image Transformation: Transforming the processed images, which includes filtering out black images, cropping and resizing the images,  and then saving them as PyTorch tensors for further processing.
  4. Sanity Check: Checking if the lengths of T1 and T2 images are equivalent, removing any mismatching files, and providing a count of slices per subject.
  
  These classes include are:
  
  * Process: Handles the overall processing of a dataset. It fetches data from the API, downloads and extracts the NIfTI files, applies HD-BET, and transforms the images.
  * DownloadData: Handles the downloading of data from different datasets. It makes sure the correct directories are created, and initiates the processing of each dataset.
  * HDBETProcessor: Applies the HD-BET tool on the input NIfTI images. It can also apply a mask to the images if provided.
  * TransformImage: Processes the NIfTI images, filters out the black images, crops, resizes and normalizes the images, and saves them as PyTorch tensors.
  * Sanity_Check: Checks if there is a one-to-one correspondence between T1 and T2 images for each subject. It also provides the functionality to count the number of slices per subject.
![UML Diagram from Data processing](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/Get_Process_Data.svg)

- `cycle/Nets.py`:Nets.py contains the implementation of three crucial classes for a Generative Adversarial Network (GAN) based on the CycleGAN architecture.It is primarily composed of the following classes:

  * ResBlock: is a class that represents a residual block, a crucial component of the generator model in our GAN. It is a common tool in  deep learning models to overcome the problem of vanishing gradients. It enables training deep networks by allowing gradient information to propagate through many layers. This block consists of two convolutional layers with instance normalization and ReLU (Rectified Linear Unit) activation.
  * Generator: is a class that represents the generator model in our GAN. This model takes an image from one domain as input and generates a corresponding image in the target domain. It is composed of an initial convolutional block, a series of ResBlocks for deep feature transformation, and a final convolutional block to generate the output image. Also, the generator employs instance normalization and uses a Tanh activation function in the output layer.
  * Discriminator: is a class that represents the discriminator model in our GAN. The purpose of this model is to classify whether a given image is real (from the dataset) or fake (generated by the generator). It is built as a PatchGAN, which means it classifies individual patches of the input image as real or fake. The advantage of a PatchGAN discriminator is that it can model high-frequency structures better than a standard GAN discriminator.
![UML Diagram from Networks](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/Nets.svg)


- `cycle/DataMod.py`: This script contains three classes used for loading and transforming the dataset for the CycleGAN model. These classes are as follows:
  * MRIImageAugmentation: This class applies random transformations to an image. The transformations include rotation, random crop, horizontal flip, and random Gaussian noise. It takes a dictionary of transformation parameters and an image sample to generate random noise.
  * CustomDataset: This class represents a dataset that contains pairs of images (T1 and T2). It provides methods to retrieve the images, apply transformations, and access their paths. The dataset can be multiplied by a factor and can include a proportion of augmented samples.
  * CycleGANDataModule: This class prepares and sets up the data for training, validation, and testing the CycleGAN model. It allows for customization of batch size, number of workers, train-test-validation split ratios, random seed, factor of data multiplication, proportion of augmented samples, and transformation parameters.
![UML Diagram from Data Module](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/DataMod.svg)

- `cycle/HyperOptim.py`: Python script that performs hyperparameter optimization using the Optuna library. This script is designed to work with PyTorch Lightning and specifically CycleGAN, which is an architecture used for image-to-image translation tasks.The script consists of two main classes: PyTorchLightningPruningCallback and HyperParametrization.

   * PyTorchLightningPruningCallback: It extends the functionality of PyTorchLightningPruningCallback and Callback classes from the PyTorch Lightning library. It is used for handling pruning in PyTorch Lightning trials.

    * HyperParametrization class performs hyperparameter optimization using the Optuna library. It searches for the best set of hyperparameters for a given function. It  creates a logging system using Python's built-in logging library. This logs information from both Python and Optuna during the optimization process.

      The objective method of HyperParametrization defines the objective function used for hyperparameter optimization. It trains a CycleGAN model using the hyperparameters generated for a given trial and returns the value of the metric being optimized.

      The Gen_HyperPar method generates the hyperparameters for a given trial. It suggests values for each hyperparameter based on the trial and the range of possible values specified for each hyperparameter.
![UML Diagram from Hyperparameter Optimization](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/HyperOptim.svg)


- `cycle/Set_FolderStructure.py`:  it helps in organizing your project by creating a predefined folder structure. This script is particularly useful when you need to consistently create several directories for different purposes.

    * DirectoryCreator class:  provides a method for creating multiple directories. It is designed to create directories for T1, T2, Model, CheckPoint, CheckPoint_Opt, GIF, and BestModel.
![UML Diagram from Set Folder Structure](https://github.com/agustinroviraquezada/MRI_T1_T2_CycleGAN/blob/main/docs/Set_FolderStructure.svg)

