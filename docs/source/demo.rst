demo
=====

.. _synthesis

Synthesis an HealPix data
-------------------------

Release with the **foscat** package the script **demo.py** shows how to use this package to generate realisation of the Venus surface having the same Scattering coefficients than the data.

To run the demo, choose the nside:

.. code-block:: console

   > python demo.y -n=64 -c -S=146 -o=MyVenus

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

This block of code defines and calls the ``main()`` function, which is the entry point of the program. Inside the ``main()`` function, the following steps are performed:

* Import all needed packages
  
.. code-block:: console
		
  import numpy as np
  import os, sys
  import matplotlib.pyplot as plt
  import healpy as hp
  import getopt

  #=================================================================================
  # INITIALIZE FoCUS class
  #=================================================================================
  import foscat.Synthesis as synthe
  
* First, several checks are performed to validate the input parameters, such as the resolution nside and the size of the kernel KERNELSZ. If the input parameters are not valid, the program exits with an error message.

.. code-block:: console
		
    def main():
    try:
        opts, args = getopt.getopt(sys.argv[1:], "n:cS:s:xpgkd:o:K", \
                                   ["nside", "cov","seed","steps","xstat","p00","gauss","k5x5","data","out","k128"])
    except getopt.GetoptError as err:
        # print help information and exit:
        print(err)  # will print something like "option -a not recognized"
        usage()
        sys.exit(2)

    cov=False
    nside=-1
    nstep=1000
    docross=False
    dop00=False
    dogauss=False
    KERNELSZ=3
    dok128=False
    seed=1234
    outname='demo'
    data="Venus_256.npy"
    instep=16

    for o, a in opts:
        if o in ("-c","--cov"):
            cov = True
        elif o in ("-n", "--nside"):
            nside=int(a[1:])
        elif o in ("-s", "--steps"):
            nstep=int(a[1:])
        elif o in ("-S", "--seed"):
            seed=int(a[1:])
            print('Use SEED = ',seed)
        elif o in ("-o", "--out"):
            outname=a[1:]
            print('Save data in ',outname)
        elif o in ("-d", "--data"):
            data=a[1:]
            print('Read data from ',data)
        elif o in ("-x", "--xstat"):
            docross=True
        elif o in ("-g", "--gauss"):
            dogauss=True
        elif o in ("-k", "--k5x5"):
            KERNELSZ=5
        elif o in ("-K", "--k128"):
            KERNELSZ=128
            instep=7
        elif o in ("-p", "--p00"):
            dop00=True
        else:
            assert False, "unhandled option"

    if nside<2 or nside!=2**(int(np.log(nside)/np.log(2))) or (nside>256 and KERNELSZ<=5) or (nside>2**instep and KERNELSZ>5) :
        print('nside should be a power of 2 and in [2,...,256] or [2,...,%d] if -K|-k128 option has been choosen'%(2**instep))
        usage()
        exit(0)

    print('Work with nside=%d'%(nside))
  
* The code then imports the appropriate module (``foscat.scat`` or ``foscat.scat_cov`` depending on whether cov is True or False.
  
.. code-block:: console
		
    if cov:
        import foscat.scat_cov as sc
        print('Work with ScatCov')
    else:
        import foscat.scat as sc
        print('Work with Scat')
	
* A scratch path is defined where intermediate data is stored.
  
.. code-block:: console
    #=================================================================================
    # DEFINE A PATH FOR scratch data
    # The data are storred using a default nside to minimize the needed storage
    #=================================================================================
    scratch_path = '../data'
    
* The ``dodown()`` function is defined to reduce the size of the input data using averaging.
  
.. code-block:: console
		
    #=================================================================================
    # Function to reduce the data used in the FoCUS algorithm
    #=================================================================================
    def dodown(a,nside):
        nin=int(np.sqrt(a.shape[0]//12))
        if nin==nside:
            return(a)
        return(np.mean(a.reshape(12*nside*nside,(nin//nside)**2),1))
	
* The input data is loaded and reduced in size using the ``dodown()`` function.
  
.. code-block:: console
		
    #=================================================================================
    # Get data
    #=================================================================================
    im=dodown(np.load(data),nside)
    
* A random noise map is generated with the same power spectrum as the input data. The input map is in nested ordering.
  
.. code-block:: console
		
    #=================================================================================
    # Generate a random noise with the same coloured than the input data
    #=================================================================================

    idx=hp.ring2nest(nside,np.arange(12*nside*nside))
    idx1=hp.nest2ring(nside,np.arange(12*nside*nside))
    cl=hp.anafast(im[idx])

    if dogauss:
        np.random.seed(seed+1)
        im=hp.synfast(cl,nside)[idx1]
        hp.mollview(im,cmap='jet',nest=True)
        plt.show()

    np.random.seed(seed)
    imap=hp.synfast(cl,nside)[idx1]

* Initialize the ``sc.funct()`` function.
  
.. code-block:: console

    lam=1.2
    if KERNELSZ==5:
        lam=1.0

    r_format=True
    if KERNELSZ==128:
        r_format=False
    #=================================================================================
    # COMPUTE THE WAVELET TRANSFORM OF THE REFERENCE MAP
    #=================================================================================
    scat_op=sc.funct(NORIENT=4,          # define the number of wavelet orientation
                     KERNELSZ=KERNELSZ,  #KERNELSZ,  # define the kernel size
                     OSTEP=-1,           # get very large scale (nside=1)
                     LAMBDA=lam,
                     TEMPLATE_PATH=scratch_path,
                     slope=1.0,
                     gpupos=0,
                     use_R_format=r_format,
                     all_type='float64',
                     SHOWGPU=True,
                     nstep_max=instep)
		     
* The Wavelet Scattering Coefficients of the input data are computed using the ``sc.funct()`` function from the imported module.
  
.. code_block:: console
		
     if docross:
        refX=scat_op.eval(im,image2=im,Imaginary=True)
    else:
        refX=scat_op.eval(im)
		 
* A loss function is defined based on the difference between the synthesized and original maps. This loss function is use to initialize a class  ``synthe.Loss`` that will be given to the synthesise function.

.. code-block:: console
		
    #=================================================================================
    # DEFINE A LOSS FUNCTION AND THE SYNTHESIS
    #=================================================================================

    def lossX(x,scat_operator,args):

        ref = args[0]
        im  = args[1]
        #ip0 = args[2]

        if docross:
            learn=scat_operator.eval(im,image2=x,Imaginary=True)
        else:
            learn=scat_operator.eval(x)

        if dop00:
            loss=scat_operator.bk_reduce_mean(scat_operator.bk_square(ref.P00[0,0,:]-learn.P00[0,0,:]))
        else:
            loss=scat_operator.reduce_sum(scat_operator.square(ref-learn))


        return(loss)

    if docross:
        refX=scat_op.eval(im,image2=im,Imaginary=True)
    else:
        refX=scat_op.eval(im)

    loss1=synthe.Loss(lossX,scat_op,refX,im)
    
* The synthesis is performed using the ``synthe.Synthesis()`` function from the foscat.synthe module.

.. code-block:: console
		
  sy = synthe.Synthesis([loss1])
    #=================================================================================
    # RUN ON SYNTHESIS
    #=================================================================================


    omap=sy.run(imap,
                DECAY_RATE=0.9995,
                NUM_EPOCHS = nstep,
                LEARNING_RATE = 0.03,
                EPSILON = 1E-15)
		
* The output data is saved to files.
  
.. code-block:: console
		
    #=================================================================================
    # STORE RESULTS
    #=================================================================================
    if docross:
        start=scat_op.eval(im,image2=imap)
        out =scat_op.eval(im,image2=omap)
    else:
        start=scat_op.eval(imap)
        out =scat_op.eval(omap)

    np.save('in_%s_map_%d.npy'%(outname,nside),im)
    np.save('st_%s_map_%d.npy'%(outname,nside),imap)
    np.save('out_%s_map_%d.npy'%(outname,nside),omap)
    np.save('out_%s_log_%d.npy'%(outname,nside),sy.get_history())

    refX.save('in_%s_%d'%(outname,nside))
    start.save('st_%s_%d'%(outname,nside))
    out.save('out_%s_%d'%(outname,nside))

    print('Computation Done')
    sys.stdout.flush()
    
Overall, this code performs a wavelet-based scattering transform image synthesis using the **FoCUS** algorithm. The algorithm takes an input image, generates a random noise map with the same power spectrum, and then synthesizes a new image that matches the scattering coefficients of teh input image. The synthesized image is saved to a file for further analysis.

once the **demo.py** script has been run, it is possible to plot the results using the function **plotdemo.py**.

.. code-block:: console
		
   > python plotdemo.y -n=64 -c -o=MyVenus


.. _work2D

Synthesis an image
-------------------

