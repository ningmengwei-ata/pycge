.. pycge documentation master file, created by
   sphinx-quickstart on Wed May 31 12:14:09 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to pycge's documentation!
=================================

Contents:

.. toctree::
   :maxdepth: 2

Requirements
---------------

This program is written in ``python-3.x`` and depends on the ```pyomo`` <http://www.pyomo.org>`_ 
package::

    pip install pyomo
    # install pyomo dependencies
    pyomo install-extras

See `<http://www.pyomo.org/installation/>`_ for more details on installation, 
and `the documentation <http://www.pyomo.org/documentation/>`_ for info on the package.

Solvers
~~~~~~~

Small to moderately sized problems may be handled through the use of remote solvers
on the `NEOS Server <https://neos-server.org/neos/>`_, which is accessible by ``pyomo``.

To use a *local* solver, see below.

Input data
~~~~~~~~~~

Before using this program, it is neccesary to prepare your data.
All data should be stored in a single directory with no other
files, other than the ones that will be used. All files should be .csv files and
should be named as follows: type-name-.csv

For example:


A file containing the set "h" should be named::

        set-h-.csv
     
While a file containing a parameter "sam" should be named::

        param-sam-.csv


Installation
------------

TODO: installation instructions


Quick Start
------------

A quick summary of a standard workflow::

     # create model definition object
     testdef = ModelDef()
     # create pycge object
     testcge = PyCGE(testdef)
     # add data
     testcge.load_data(path/to/data)
     # create base instance
     testcge.model_instance()
     # calibrate base instance
     testcge.model_calibrate(solver, mgr='') 
     # create sim instance to modify
     testcge.model_sim() 
     # modify sim instance
     testcge.model_modify_instance(...) # modify some value
     testcge.model_modify_instnace(...) # modify another value
     # solve sim instance
     testcge.model_solve(solver, mgr=') 
     # compare base and sim equilibria
     testcge.model_compare()

Read on for more details.

Getting Started
---------------
Define a model::

    test_def = ModelDef()

Create a model from a ``ModelDef``::

    test_cge = PyCGE(test_def)

Load the data::

    test_cge.model_data(directory/that/contains/data/files)


Instantiate the model::

    test_cge.model_instance(NAME, INDEX)

Note: In this step it is neccesary to fix the numeraire. For example ``test_cge.model_instance('pf','CAP')`` would fix
the index "CAP" of the variable "pf" at it's current value.

Also note: Some solvers (IPOPT for example) delete fixed variables. Please make sure the solver chosen can handle a fixed
variable in an appropriate way.

Calibrate the Base Model
------------------------

The initial call to ``model_instance()`` creates a ``base`` model.
To calibrate the ``base`` model::

     test_cge.model_calibrate(solver, mgr='')

This call solves the ``base`` model. Where ``solver`` is the solver the user would like to use (ex. "minos") 
and ``mgr`` is the server the user would like to use (ex. "neos").

A successful calibration should return
equilibrium values equal to the initial values.

If calibration fails, check your data and your model definition.

Equilibrium Comparative Statics
-------------------------------

Once the ``base`` model is calibrated, we can simulate policy changes or shocks to
our economy and perform comparative statics (i.e., comparing the equilibria with and
without the policy change). 

First, create a ``sim`` instance::

     test_cge.model_sim()

This step creates a second instance called ``sim`` by copying the ``base`` instance. 

Now you can explore a policy change relative to ``base`` by modifying the ``sim`` 
instance (changing a parameter value or fixing a variable). 

To modify an instance::

    test_cge.model_modify_instance(NAME, INDEX, VALUE, fix=True, undo=False)

where: 

- ``NAME`` is a string (the name of the ``Var`` or ``Param`` to be modified) 
- ``INDEX`` is a string (the index where the modification will be made) 
- ``VALUE`` is numeric (the modification)

For example, to modify a variable ``X`` so that ``X[i] == 0``::

     test_cge.model_modify_instance('X', 'i', 0)

Note that the default for modifying a variable means *fixing* it at some value. 
To unfix a variable simply pass ``fix=False`` into the function call. 

Also note that in order to modify a two-dimensional parameter, it must be surrounded by parenthesis.
For example::
    test_cge.model_modify_instance('F0',('CAP','BRD'),0)

While modifying a scalar parameter, simply pass in ``None`` as the ``INDEX``
For examples::
	test_cge.model_modify_instance('F0',None,0)

To "undo" a modification simply pass in ``undo=True``::

	test_cge.model_modify_instance(NAME,INDEX,None, fix= True, undo=True)

Note that this will simply return it to the previous value. Hence, it is reccomended that the user always 
undoes a modification before making another to keep the "original" value saved

To solve the ``sim`` instance, e.g., 
using the Minos solver on `NEOS <neos-server.org/neos>`_::

    test_cge.model_solve("minos", "neos")

**Remark**: Other nonlinear solvers available on NEOS include
- ``"conopt" 
- "ipopt" 
- "knitro"``

Note: the default for solving a ``base`` or ``sim`` instance is to use a local solver (i.e. mgr='').

Viewing an Instance or Results
------------------------------

To export anything::
    
    	test_cge.model_postprocess(object_name="", verbose="", base=True)
    	
(Note: by default this will deal with ``base`` instance, results, etc. Pass in ``base=False`` to deal with ``sim`` objects)

- To output display of instance
	- ``object_name="instance"``
        	- ``verbose="print"`` to print instance display
        	- ``verbose="directory/name/"`` to export instance display in a file
        
- To output display of results
	- ``object_name="results"``
        	- ``verbose="print"`` to print results display
        	- ``verbose="directory/name/"`` to export results display in a file

- To output variables as a .csv file
	- ``object_name="vars"``
        	- ``verbose="directory/name/"`` to export each variable in a file

- To output objective as a .csv file
	- ``object_name="obj"``
        	- ``verbose="directory/name/"`` to export obj in a file

- To output dilled instance object (can later be loaded)
	- ``object_name="dill_instance"``
        	- ``verbose="directory/name/"`` to export dilled instance object in a file

- To output comparative statics (this shows *difference* and *percentage chnaged* in equilibrium values between ``base`` and ``sim``)::
	- ``object_name="compare"``
		-``verbose="print"`` to print the comparision
        	- ``verbose="directory/name/"`` to export the comparision in a file (The user should name this file specific to what the comparision is)

- To output parameters (Note: this shows parameter name, value, and doc)
	- ``object_name="params``
		- ``verbose=""`` (This is the default)
        
Updating
---------------

To load an instance object(with results attached) back in as ``base`` instance (set base=False to load as ``sim`` instance)::

    test_cge.model_load_instance(pathname = pathname/of/file/to/load, base=True)

Two or More Simulations
-------------------------

Currently, an object of class ``PyCGE`` can only have two instances associated with it,
``base`` and ``sim``. 

Calling ``model_sim`` always creates a new ``sim`` based on the ``base`` instance.
If you call ``model_modify_instance`` *after* solving ``sim``, you will overwrite the 
previous ``sim`` instance. 

If you want to analyze multiple policy changes or shocks (i.e., you want to analyze multiple
``sim`` instances) **and** you want to keep each ``sim`` instance, the easiest thing to do 
is to copy your existing object and perform the new simulation on the copy::

     import copy
     copy_cge = deepcopy(test_cge)
     copy_cge.model_sim()
     copy_cge.model_modify_instance()
     # etc.

Now you can perform comparative statics on two ``sim`` instances, one associated with the
``test_cge`` object and one associated with the ``copy_cge`` object. Both have the same 
``base`` instance but potentially differ in the ``sim`` instance. 

Local Solvers
--------------

TODO: Add info on installing Ipopt, etc.
Remember to note that Spyder has trouble running local solver.

Working With Model Definitions
------------------------------

A model definition is contained within a ``ModelDef`` class, and should be imported along with
``PyCGE``::

    import pycge
    from model_def import ModelDef

You may work with multiple ``ModelDef`` classes, e.g., ``ModelDef1``, ``ModelDef2``, and so on.
You may also edit an existing ``ModelDef`` and re-load it using ``importlib``::

    importlib.reload(ModelDef)

Order of Operations
-------------------
None of these can be done before all previous steps are completed 

Note: Messages will be displayed if the user tries to go out of order and will help guide them. 

1. Create ``base`` instance
2. OPTIONAL: Modify ``base`` instance (must return to this step each time the user modifies ``base`` instance)
3. Solve ``base`` instance (cannot solve again unless the ``base`` instance is modified. see Step 2)
4. Create ``sim`` instance
5. OPTIONAL: Modify ``sim`` instance (must return to this step each time the user modifies ``sim`` instance)
6. Solve ``sim`` instance (cannot solve again unless the ``sim`` instance is modified. see Step 5)


Examples
--------
These are example scripts to show the basic setup, workflow, and some of the basic capabilities of PyCGE.

This first example will compare two policy changes.
Abolishing Tarrifs (to incentivize imports) Vs. Abolishing Production Taxes (to incentivize production)
At the end of the script a welfare measure for each policy will be printed out to help decision makes evaluate the value of each.::

    # Choose which model to look at, and create a ModelDef object
    std_model = StdModelDef()
    
    # Create a PyCGE object and load the ModelDef object into it
    testcge = PyCGE(std_model)
    
    # Load the data by passing the path to the directory that contains it
    testcge.model_data('../data/stdcge_data_dir')
    
    # Create a `base` instance and pass in a variable and index to fix the numeraire
    testcge.model_instance('pf', 'CAP')
    
    # Calibrate the `base` instance
    testcge.model_calibrate('minos','neos')
    
    # Create a `sim` instance. This, right now, is exactly the same as the `base` instance
    testcge.model_sim()
    
    
    # Now a copy of the first PyCGE object can be made
    # This will save the user having to perform all previous steps again
    import copy
    copycge = copy.deepcopy(testcge)
    
    
    # This is the first policy change
    # Modify the sim instance to set import tarrifs to 0 for each good
    testcge.model_modify_sim('taum','BRD',0)
    testcge.model_modify_sim('taum','MLK',0)
    
    # Solve the `sim` instance
    testcge.model_solve('minos','neos')
    
    # Compare the equilibrium values between `base` and `sim`
    testcge.model_postprocess('compare','print')
    
    
    
    
    # This is the second policy change
    # Modify the sim instance to set production taxes to 0 for each good
    copycge.model_modify_sim('tauz','BRD',0)
    copycge.model_modify_sim('tauz','MLK',0)
    
    # Solve the `sim` instance
    copycge.model_solve('minos','neos')
    
    # Compare the equilibrium values between `base` and `sim`
    copycge.model_postprocess('compare','print')
    
    
    
    
    # This is an example of how one can use results to perform calculations such as computing EV
    def model_welfare(PyCGE):
        # Solve for Hicksian equivalent variations
        print('\n----Welfare Measure----')
        ep0 = (value(PyCGE.base.obj)) /prod((PyCGE.base.alpha[i]/1)**PyCGE.base.alpha[i] for i in PyCGE.base.alpha)
        ep1 = (value(PyCGE.sim.obj)) / prod((PyCGE.base.alpha[i]/1)**PyCGE.base.alpha[i] for i in PyCGE.base.alpha)
        EV = ep1-ep0
        
        print('Hicksian equivalent variations: %.3f' % EV)
    
    
    # Time to see the value of each policy change
    print('----------------------------------------')
    print("\nAbolish tarrifs")
    model_welfare(testcge)
    
    print("\nAbolish taxes")
    model_welfare(copycge)

This second example shows how easy it is to view and modify parameters.::

    # Choose which model to look at, and create a ModelDef object
    spl_model = SplModelDef()
    
    
    # Create a PyCGE object and load the ModelDef object into it
    testcge = PyCGE(spl_model)
    
    
    # Load the data by passing the path to the directory that contains it
    testcge.model_data('../data/splcge_data_dir')
    
    
    # Create a `base` instance and pass in a variable and index to fix the numeraire
    testcge.model_instance('pf', 'CAP')
    
    
    # Calibrate the `base` instance
    testcge.model_calibrate('minos','neos')
    
    
    # This allows the user to see all the params, their values, and what their docs
    print('#===========original============#')
    testcge.model_postprocess('params')
    
    # Modify a parameters
    testcge.model_modify_base('X0','MLK',0)
    
    # View the params again, notice X0[MLK] is now set to 0
    print('#===========X0[MLK] set to 0============#')
    testcge.model_postprocess('params')
    
    # By passing in `undo=True` a user can undo their last change
    testcge.model_modify_base('X0','MLK',None,undo=True)
    
    # View all params again. Notice that X0[MLK] was restored to its original value 
    print('#===========after the "undo"============#')
    testcge.model_postprocess('params')

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

