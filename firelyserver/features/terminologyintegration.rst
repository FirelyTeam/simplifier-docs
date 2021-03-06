.. _feature_terminologyintegration:

External terminology services
=============================

To be able to use more elaborate terminology support, 
we provide the option to integrate external FHIR terminology services with Firely Server. 

Firely Server can redirect the following terminology operations to external services:

* $validate-code
* $expand
* $lookup
* $translate
* $subsumes

.. warning:: The TerminologyIntegration plugin does not work together with the Firely Server Terminology plugin. 

Make sure to add the plugin to the PipelineOptions in appsettings in order to make use of the ``TerminologyIntegration`` plugin.

.. code-block:: JavaScript

    "PipelineOptions": {
        "PluginDirectory": "./plugins",
        "Branches": [
          {
            "Path": "/",
            "Include": [
              "Vonk.Core",
              "Vonk.Fhir.R3",
              "Vonk.Fhir.R4",
              //"Vonk.Fhir.R5",
              "Vonk.Repository.Sql.SqlVonkConfiguration",
              "Vonk.Repository.Sqlite.SqliteVonkConfiguration",
              "Vonk.Repository.MongoDb.MongoDbVonkConfiguration",
              "Vonk.Repository.CosmosDb.CosmosDbVonkConfiguration",
              "Vonk.Repository.Memory.MemoryVonkConfiguration",
              "Vonk.Subscriptions",
              "Vonk.Smart",
              "Vonk.UI.Demo",
              "Vonk.Plugin.DocumentOperation.DocumentOperationConfiguration",
              "Vonk.Plugin.ConvertOperation.ConvertOperationConfiguration",
              "Vonk.Plugin.BinaryWrapper",
              "Vonk.Plugin.MappingToStructureMap.MappingToStructureMapConfiguration",
              "Vonk.Plugin.TransformOperation.TransformOperationConfiguration",    
              "Vonk.Plugin.Audit",          
              "Vonk.Plugins.TerminologyIntegration",
              "Vonk.Plugin.Mapping"          
            ],
            "Exclude": [
              "Vonk.Subscriptions.Administration"
            ]
          }, ...etc...

.. note::
    Adding Vonk.Plugins.TerminologyIntegration means you cannot use Vonk.Plugins.Terminology. Firely Server will log a warning if you have both      plugins included in the appsettings. 
    
Options
-------

You can enable the integration with one or more external terminology services by setting the required options in the appsettings file. 
For each terminology service you can set the following options:

    :Name: The name of the terminology service.
    :Endpoint: The endpoint url where Firely Server can redirect the requests to.
    :Username: If the terminology service uses Basic Authentication, you can set the required username here. 
    :Password: If the terminology service uses Basic Authentication, you can set the required password here.
    :Priority: The priority of the terminology service. If multiple Terminology services could be used for a request, Firely Server will use the priority to select a service. Terminology services are arranged in a descending order: so 1 is first priority, 2 is next in priority and so on.
    :PreferredSystem: If a request is directed at a specific code system, Firely Server will choose this terminology server over other available services.
    :SupportedInteractions: The operations supported by the terminology service. Firely Server will only select this service if the operation is in this list.
    :SupportedResources: The resources supported by the terminology service.


A sample Terminology section in the appsettings can look like this:

.. code-block:: JavaScript

    "Terminology": {
        "TerminologyServices": [
          {
            "Name": "Loinc",
            "Endpoint": "https://fhir.loinc.org/",
            "Username": <username>,
            "Password": <password>,
            "Priority": "10",
            "PreferredSystem": [ "http://loinc.org" ],
            "SupportedInteractions": [ "validate-code", "expand", "translate", "lookup" ],
            "SupportedResources": [ "CodeSystem", "ValueSet", "ConceptMap", "Questionnaire" ]
          },
          {
            "Name": "Ontoserver",
            "Endpoint": "https://ontoserver.csiro.au/stu3-latest/",
            "Username": "",
            "Password": "",
            "PreferredSystem": [ "http://snomed.info/sct", "http://hl7.org/fhir", "http://csiro.au/" ],
            "Priority": "20",
            "SupportedInteractions": [ "validate-code", "expand", "translate", "lookup", "subsumes" ],
            "SupportedResources": [ "CodeSystem", "ValueSet", "ConceptMap" ]
          }
        ]
      }, ...etc...
  
This means if you execute a terminology operation request, Firely Server will validate the request, redirect it to the preferred terminology service and finally return the result. 
