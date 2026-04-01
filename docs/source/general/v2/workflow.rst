Workflow
==============

The iDDS has implemented a workflow architecture to support new use cases. The Workflow
consists of Work (Transformation) and Condition, where Condition can be used to implement
DAG support. New use cases can be implemented by inheriting the Work class and implementing
the required hook functions.

.. image:: ../../images/v2/architecture_daemon_flow.png
      :alt: iDDS Architecture

DictClass
~~~~~~~~~

DictClass is designed to automatically convert all attributes of the class to a JSON dictionary
and convert it back from the JSON dictionary to a class instance. The Workflow and Work classes are
inherited from this class. In this way, the Workflow and Work can be transmitted from the client to
the REST service and be saved to the database in JSON format. When reading, this JSON format can be
converted back to a Workflow or Work instance, which simplifies handling in subsequent steps.

Workflow
~~~~~~~~

.. image:: ../../images/v2/workflow_subworkflow.jpg
         :width: 45%
         :alt: Sub Workflow

.. image:: ../../images/v2/loopworkflow.jpg
         :width: 45%
         :alt: Loop Workflow

A Workflow is designed to manage multiple Works (Transformations) and is a subclass of DictClass.
By using Conditions, a Workflow can represent and manage DAG-style execution.

SubWorkflow
~~~~~~~~~~~~~~~

A workflow can be added as a subworkflow of another workflow. In this case, it can be regarded as a Work.
However, executing this Work will generate new Works at runtime.

LoopWorkflow
~~~~~~~~~~~~~~~

When a loop condition is added to a workflow, the workflow can be executed repeatedly.

Work
~~~~

A Work, a subclass of DictClass, is a transformation. New use cases can be implemented by
inheriting the Work class.

1. Functions that need to be overridden for a Transformer:

        a. get_input_collections: poll DDM to get the status and metadata of the collections.
        b. get_new_input_output_maps(registered_input_output_maps): registered_input_output_maps is provided
           by iDDS with contents registered in the iDDS database. This function should return maps between
           new inputs and outputs.
        c. create_processing(input_output_maps): create a processing with maps between inputs and outputs.
        d. syn_work_status(registered_input_output_maps): registered_input_output_maps is provided
           by iDDS with contents registered in the iDDS database. It works to update the Work.status to
           Transforming, Finished, SubFinished, or Failed based on the status of all outputs in
           registered_input_output_maps.

2. Functions that need to be overridden for a Carrier:

        a. submit_processing: implement how to submit the processing.
        b. poll_processing_updates: implement how to poll the processing status.

WorkflowManager
~~~~~~~~~~~~~~~

It works to automatically convert a workflow to an iDDS request (the workflow will be converted
to a JSON dictionary by DictClass) and send the request to the iDDS RESTful service.

