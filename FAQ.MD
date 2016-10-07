# Frequently Asked Questions

## Table of Contents
* [How to specify server host and port ?](#how-to-specify-server-host-and-port-)
* [What are the machine learning frameworks and algorithms supported by the server ?](#what-are-the-machine-learning-frameworks-and-algorithms-supported-by-the-server-)
* [Getting this error: ‘dd_msg’: ‘src/caffe/syncedmem.cpp:65 / Check failed (custom): (error) == (cudaSuccess)’](#getting-this-error-dd_msg-srccaffesyncedmemcpp65--check-failed-custom-error--cudasuccess)
* [Getting this error: “using template while network weights exist, remove ‘template’ from ‘mllib’ or would you like to ‘finetune’ instead”](#getting-this-error-using-template-while-network-weights-exist-remove-template-from-mllib-or-would-you-like-to-finetune-instead)
* [Getting this error: “negative supervised discrete target (e.g. wrong use of label_offset ?)”](#getting-this-error-negative-supervised-discrete-target-eg-wrong-use-of-label_offset-)
* [How to import a model from Caffe into DeepDetect](#how-to-import-a-model-from-caffe-into-deepdetect)

## How to specify server host and port ?

	./dede -host yourhost -port 8888

and replace yourhost and 8888 by your own values. Default values are localhost and 8080 respectively.

## What are the machine learning frameworks and algorithms supported by the server ?

Two libraries are supported at the moment:
- Caffe, one of the most powerful deep neural network frameworks. Caffe has support for many neural architectures, including logistic regression, multi-layer perceptron, convolutional networks, recurrent nets, etc…
- XGBoost one of the most regarded machine learning frameworks for a wide range of applications. Gradient boosted trees are a form of decision trees.

DeepDetect uses a custom version of Caffe, with careful improvements while integrating all changes made to the original version, with very little delay.

DeepDetect uses the most up to date version of XGBoost at all time.

## Getting this error: ‘dd_msg’: ‘src/caffe/syncedmem.cpp:65 / Check failed (custom): (error) == (cudaSuccess)’

Most likely your operations are requiring more memory than what is available on your GPU. Try setting gpu:false everywhere to run on CPU. If this goes through, this is a confirmed GPU memory error.

To avoid this error, try lowering the batch_size as much as possible or feasible for your need.

## Getting this error: “using template while network weights exist, remove ‘template’ from ‘mllib’ or would you like to ‘finetune’ instead”

The deepdetect server protects existing neural net models and architectures from being overriden by a template, called from API. This protects the Caffe prototxt files to be modified when neural net trained weights are in place, which would lead to potential inconsistencies.

This protection is useful in production systems, but can be annoying in the development phase, here is how to proceed safely:
- if you are re-creating a service from a model repository after creating it with a built-in template, simply remove the template parameter from the PUT /services/yourservice call
- if you are changing template and don’t mind recomputing the training and testing sets, simply clear up everything with:
	curl -X DELETE "http://localhost:8080/services/yourservice?clear=full"
- if you are changing template and don’t want to recompute the training set (e.g. useful on large image datasets), clear up the models:

	curl -X DELETE "http://localhost:8080/services/yourservice?clear=lib"

## Getting this error: “negative supervised discrete target (e.g. wrong use of label_offset ?)”

DeepDetect requires discrete classification labels to start from 0 to your problem’s number of classes in increment of 1, e.g. 0, 1, 2, 3, … This is mostly to harmonize among machine learning back-ends and because Softmax requires positive integers. If your dataset labels start from 1, you’d have to use label_offset:-1 or make modification to your dataset.

Notes:
- If your classes are strings like cats, dogs, etc…, DeepDetect does the conversion automatically, there’s no issue
- If your labels are -1 and 1 as can be common in some binary classification problems, you need to modify your dataset, e.g. to get 0 and 1.

## How to import a model from Caffe into DeepDetect

Importing a model is done by moving the Caffe neural net definition files .prototxt and optionnally the model weights file .caffemodel into a DeepDetect model repository. In general, the .prototxt files need the following modifications:

- the .prototxt file definition for training and testing need to have at least the test input as a MemoryDataLayer, e.g.:
	layer {
	  name: "googlenet"
	  type: "MemoryData"
	  top: "data"
	  top: "label"
	  memory_data_param {
	  batch_size: 32
	  channels: 3
	  height: 224
	  width: 224
	  }
	  include {
	  phase: TEST
	 }
	}

In general this is a trivial modification to most of the existing Caffe model definitions. The built-in Caffe templates can be used as examples.

    the .prototxt definition file for prediction needs to be renamed deploy.prototxt and its input layer must use a MemoryDataLayer
    the solver .prototxt file must end with _solver.prototxt

DeepDetect documentation
