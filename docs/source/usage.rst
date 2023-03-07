Usage
=====

.. _installation:

Installation
------------

To use Foscat, first install it using pip:

.. code-block:: console

   >pip install foscat

Compute scattering Covariance
-----------------------------

To compute scattering covariance,
you can use the ``foscat.scat_cov`` python class:

.. autofunction:: foscat.scat_cov


For example:

>>> import foscat.scat_cov as sc
>>> scat_op=sc.funct()
>>> im=np.load('Venus.py')
>>> sCoef=scat_op.eval(im)
>>> sCoef.plot()

