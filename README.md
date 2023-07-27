# MSIpredictART

## Getting started

This tool relies on the [Esmraldi](https://github.com/fgrelard/Esmraldi) library. Start by downloading and installing it. Then, follow the [Getting started](https://esmraldi.readthedocs.io/en/latest/user/gettingstarted.html#setting-up-esmraldi-cross-platform) instructions. Finally, **install Esmraldi locally** by running the following command, in the Esmraldi folder: 

```bash
pip install .
```

You can now use the tools in MSIpredictART.

## Usage


The core of the Esmraldi library is contained in the `esmraldi`
directory.

The `examples` directory contains various scripts to process MSI data.

The `gui` directory is everything GUI related (views and controllers).

### General script usage

Several examples are available for each step of the fusion workflow.
They are located in the `examples` directory in the Esmraldi repository.
These examples include code embedded in the [Quickstart
tutorial](quickstart.ipynb) and [Advanced
example](advanced_example.ipynb), and make it easier to reuse the code.

The general command is the following: :

    python -m examples.<module_name> [--options] [--help]

We strongly advise to refer to the output given by the `help` argument
before using these examples; it describes the example file and its
parameters.

The scripts generally expect arguments, usually the input image, as well
as other parameters depending on the script. The *\--help* argument
lists the arguments and provides a short description of them.

The path to the images and output must be given as a path (e.g.
**D:\\Data\\image.imzML** for Windows or **/home/user/Data/image.imzML**
for Linux).

Arguments have a full version with two dashes (e.g. *\--input* and
generally a shorter version with a single dash (e.g. *-i*).

### Common errors


    No module named xxx

Make sure the path to the module is correct. Modules should be called by
prefixing their name with \"examples\".

    FileNotFoundError: [Errno 2]

The path does not exist. Make sure your path to the input is correctly
formatted. Make sure the directory exists (create a new directory
first).

    TypeError: expected str, bytes or os.PathLike object, not NoneType

Two options: a mandatory argument was omitted, or you did not provide
the correct type for the argument (float or integer numbers, character
strings\...). Check the help for more information.

## Module organization

### Supervised learning

-   `create_image_for_pls.py`: script to generate a dataset used for
    learning, with subsampling specified with *sample\_size* argument. :

        python -m create_image_for_pls -i /home/Data/dataset1/peakpicked.imzML /home/Data/dataset2/peakpicked.imzML  --regions /home/Data/dataset1/masks/resized/*.tif --regions /home/Data/dataset1/masks/resized/*.tif  --sample_size 1000 -o /home/Data/train/train_dataset.tif --normalization

-   `pls.py`: Training with LASSO or PLS. Expects either an *alpha*
    value for Lasso or *nb\_component* for PLS. Generates a model file
    (joblib extension).:

        python -m examples.pls -i /home/Data/train/train_dataset.tif -r /home/Data/train/regions/*.tif  -o /home/Data/models/model.joblib --lasso --alpha 0.002

-   `bootstrap_models.py`: Combining several Lasso or PLS on different
    trained models.:

        python -m examples.bootstrap_model -i /home/Data/models/model1.joblib /home/Data/models/model2.joblib --lasso -o /home/Data/combination.joblib

-   `evaluate_models.py`: Validation of the model, using a validation
    dataset (should be created first using `create_image_for_pls.py`). :

        python -m examples.evaluate_models -i /home/Data/models/ --validation_dataset /home/Data/validation/validation.tif --lasso 

-   `model_assign_gmm.py`: fit a gaussian mixture model (GMM) onto the
    trained model, to obtain an \"uncertain\" class. :

        python -m examples.model_assign_gmm -i /home/Data/models/model.joblib --msi /home/Data/train/train.tif --names Binder1 Binder2 Binder3 -o /home/Data/models/model_gmm.joblib

-   `pls_test.py`: Applies the previously trained model to dataset to
    get predictions. Can use a GMM, and specify a probability to assign
    to the \"Uncertain\" class (*proba* argument):

        python -m examples.pls_test -i /home/Data/models/model.joblib -t /home/Data/dataset3/peakpicked.imzML  --gmm  /home/Data/models/model_gmm.joblib --names Binder1 Binder2 Binder3 -o /home/Data/dataset3/prediction.png --proba 0.95 --normalization

-   `evaluation_prediction_confusion.py`: Get sensibility, specificity,
    precision (and more) matrices, typically for a training dataset. :

        python -m examples.evaluation_prediction_confusion -i /home/Data/models/model.joblib -t /home/Data/dataset1/peakpicked.imzML -o /home/Data/evaluation.xlsx --names Binder1 Binder2 Binder3 --normalization  --gmm /home/Data/models/model_gmm.joblib --proba 0.95

-   `compare_prediction.py`: viewer to compare predictions across
    various parameters. :

        python -m examples.compare_prediction -i /home/Data/models/ --parameters 0.01 0.02 0.03 0.04 --keys P2D3 P2F4 P2D6

-   `display_model_graph_from_dataset.py`: Display a tSNE projection of
    the Gaussian Mixture Model of the training dataset. :

        python -m examples.display_model_graph_from_dataset -i /home/Data/models/model.joblib --msi /home/Data/train/train.tif --names Binder1 Binder2 Binder3 --gmm /home/Data/models/model_gmm.joblib
