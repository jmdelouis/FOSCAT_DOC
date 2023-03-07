demo
=====

.. _synthesis

Synthesis an HealPix data
-------------------------

Release with the **foscat** package the script **demo.py** shows how to use this package to generate realisation of the Venus surface having the same Scattering coefficients than the data.

To run the demo, choose the nside:

.. code-block:: console

   (.venv) $ python demo.y -n=64 -c -S=146 -o=MyVenus

**demo.py** have lot of options:

.. code-block:: console

    > python demo.py -n=8 [-c|--cov][-s|--steps=3000][-S=1234|--seed=1234][-x|--xstat][-p|--p00][-g|--gauss][-k|--k5x5][-d|--data][-o|--out][-K|--k128]
     -n : is the nside of the input map (nside max = 256 with the default map)
     --cov (optional): use scat_cov instead of scat.
     --steps (optional): number of iteration, if not specified 1000.
     --seed  (optional): seed of the random generator.
     --xstat (optional): work with cross statistics.
     --p00   (optional): Loss only computed on p00.
     --gauss (optional): convert Venus map in gaussian field.
     --k5x5  (optional): Work with a 5x5 kernel instead of a 3x3.
     --k128  (optional): Work with 128 pixel kernel reproducing wigner matrix computation instead of a 3x3.
     --data  (optional): If not specified use Venu_256.npy.
     --out   (optional): If not specified save in *_demo_*.'

once the **demo.py** script has been run, it is possible to plot the results using the function **plotdemo.py**

   > python plotdemo.y -n=64 -c -o=MyVenus


.. _work2D

Synthesis an image
-------------------

