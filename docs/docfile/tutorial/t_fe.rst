.. _fe:

AutoGL Feature Engineering
==========================

We provide a series of node and subgraph feature engineers for 
you to compose within a feature engineering pipeline. An automatic
feature engineering algorithm is also provided.

Quick Start
-----------
.. code-block :: python

    # 1. Choose a dataset.
    from autogl.datasets import build_dataset_from_name
    data = build_dataset_from_name('cora')

    # 2. Compose a feature engineering pipeline
    from autogl.module.feature import BaseFeatureAtom,AutoFeatureEngineer
    from autogl.module.feature.generators import GeEigen
    from autogl.module.feature.selectors import SeGBDT
    from autogl.module.feature.subgraph import SgNetLSD
    # you may compose feature engineering atoms through BaseFeatureAtom.compose
    fe = BaseFeatureAtom.compose([
    GeEigen(size=32) ,
    SeGBDT(fixlen=100),
    SgNetLSD()
    ])
    # or just through '&' operator
    fe = fe & AutoFeatureEngineer(fixlen=200,max_epoch=3)

    # 3. Fit and transform the data
    fe.fit(data)
    data1=fe.transform(data,inplace=False)

List of FE atom names
---------------------
Now three kinds of feature engineering atoms are supported,namely ``generators``, ``selectors`` , ``subgraph``.You can import 
atoms from according module as is mentioned in the ``Quick Start`` part. Or you may want to just list names of atoms
in configurations or as arguments of the autogl solver. 

1. ``generators``

+---------------------------+-------------------------------------------------+
|           Atom            |                   Description                   |
+===========================+=================================================+
| ``graphlet``              | concatenate local graphlet numbers as features. |
+---------------------------+-------------------------------------------------+
| ``eigen``                 | concatenate Eigen features.                     |
+---------------------------+-------------------------------------------------+
| ``pagerank``              | concatenate Pagerank scores.                    |
+---------------------------+-------------------------------------------------+
| ``PYGLocalDegreeProfile`` | concatenate Local Degree Profile features.      |
+---------------------------+-------------------------------------------------+
| ``PYGNormalizeFeatures``  | Normalize all node features                     |
+---------------------------+-------------------------------------------------+
| ``PYGOneHotDegree``       | concatenate degree one-hot encoding.            |
+---------------------------+-------------------------------------------------+
| ``onehot``                | concatenate node id one-hot encoding.           |
+---------------------------+-------------------------------------------------+

2. ``selectors``

+----------------------+--------------------------------------------------------------------------------+
|         Atom         |                                  Description                                   |
+======================+================================================================================+
| ``SeFilterConstant`` | delete all constant and one-hot encoding node features.                        |
+----------------------+--------------------------------------------------------------------------------+
| ``gbdt``             | select top-k important node features ranked by Gradient Descent Decision Tree. |
+----------------------+--------------------------------------------------------------------------------+

3. ``subgraph``

``netlsd`` is a subgraph feature generation method. please refer to the according document.

A set of subgraph feature extractors implemented in NetworkX are wrapped, please refer to NetworkX for details.  (``NxLargeCliqueSize``, ``NxAverageClusteringApproximate``, ``NxDegreeAssortativityCoefficient``, ``NxDegreePearsonCorrelationCoefficient``, ``NxHasBridge``
,``NxGraphCliqueNumber``, ``NxGraphNumberOfCliques``, ``NxTransitivity``, ``NxAverageClustering``, ``NxIsConnected``, ``NxNumberConnectedComponents``, 
``NxIsDistanceRegular``, ``NxLocalEfficiency``, ``NxGlobalEfficiency``, ``NxIsEulerian``)

The taxonomy of atom types is based on the way of transforming features. ``generators`` concatenate the original features with ones newly generated
or just overwrite the original ones. Instead of generating new features , ``selectors`` try to select useful features and keep learned selecting methods
in the atom itself. The former two types of atoms can be exploited in node or edge level (modification upon each
node or edge feature) ,while ``subgraph`` focuses on feature engineering  in graph level (modification upon each graph feature). 
For your convenience in further development,you may want to design a new item by inheriting one of them. 
Of course, you can directly inherit the ``BaseFeatureAtom`` as well.

Create Your Own FE
------------------
You can create your own feature engineering object by simply inheriting one of feature engineering atom types ,namely ``generators``, ``selectors`` , ``subgraph``,
and overloading methods ``_fit`` and ``_transform``.

.. code-block :: python

    # for example : create a node one-hot feature.
    from autogl.module.feature.generators.base import BaseGenerator
    import numpy as np
    class GeOnehot(BaseGenerator):
        def __init__(self):
            super(GeOnehot,self).__init__(data_t='np',multigraph=True,subgraph=False) 
            # data type in mid is 'numpy',
            # and it can be used for multigraph, 
            # but not suitable for subgraph feature extraction.
        
        def _fit(self):
            pass # nothing to train or memorize

        def _transform(self, data):
            fe=np.eye(data.x.shape[0])
            data.x=np.concatenate([data.x,fe],axis=1)
            return data 

