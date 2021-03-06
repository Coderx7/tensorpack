
# DataFlow

### What is DataFlow

DataFlow is a library to build Python iterators for efficient data loading.

A DataFlow has a `get_data()` generator method,
which yields `datapoints`.
A datapoint is a **list** of Python objects which is called the `components` of a datapoint.

For example, to train on MNIST dataset, you can build a DataFlow with a `get_data()` method
that yields datapoints (lists) of two components:
a numpy array of shape (64, 28, 28), and an array of shape (64,).

### Composition of DataFlow
One good thing about having a standard interface is to be able to provide
the greatest code reusability.
There are a lot of existing DataFlow utilities in tensorpack, which you can use to compose
complex DataFlow with a long pre-processing pipeline. A common pipeline usually
would __read from disk (or other sources), apply augmentations, group into batches,
prefetch data__, etc. A simple example is as the following:

````python
# a DataFlow you implement to produce [tensor1, tensor2, ..] lists from whatever sources:
df = MyDataFlow(dir='/my/data', shuffle=True)
# resize the image component of each datapoint
df = AugmentImageComponent(df, [imgaug.Resize((225, 225))])
# group data into batches of size 128
df = BatchData(df, 128)
# start 3 processes to run the dataflow in parallel, and communicate with ZeroMQ
df = PrefetchDataZMQ(df, 3)
````
You can find more complicated DataFlow in the [ResNet training script](../examples/ResNet/imagenet-resnet.py)
with all the data preprocessing.

Unless you are working with standard data types (image folders, LMDB, etc),
you would usually want to write the base DataFlow (`MyDataFlow` in the above example) for your data format.
See [another tutorial](http://tensorpack.readthedocs.io/en/latest/tutorial/extend/dataflow.html)
for details on writing a DataFlow.

### Why DataFlow

1. It's easy: write everything in pure Python, and reuse existing utilities. On the contrary,
	 writing data loaders in TF operators is painful.
2. It's fast (enough): see [Input Pipeline tutorial](http://tensorpack.readthedocs.io/en/latest/tutorial/input-source.html)
	 on how tensorpack handles data loading.

Nevertheless, tensorpack support data loading with native TF operators as well.

### Use DataFlow outside Tensorpack
DataFlow is __independent__ of both tensorpack and TensorFlow.
You can simply use it as a data processing pipeline and plug it into any other frameworks.

To use a DataFlow independently, you will need to call `reset_state()` first to initialize it,
and then use the generator however you want:
```python
df = SomeDataFlow()

df.reset_state()
generator = df.get_data()
for dp in generator:
	# dp is now a list. do whatever
```
