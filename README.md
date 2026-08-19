# cloud
Learning Objectives
• Understand the purpose of an Azure Machine Learning workspace and its main components.
• Create an Azure Machine Learning workspace using the Azure Portal.
• Create a Compute Instance to run notebooks and training jobs.
• Explore Azure ML Studio: Notebooks, Data assets, Environments, Model registry, Endpoints, Pipelines, and Automated ML.
• Connect to the workspace programmatically using the Azure ML Python SDK v2.

#steps:
1. create a resource group
2. search azure machine learning and create workspace
3. create compute instance - cpu, DS11
4. enable SSO, disable ssh
5. launch VS code for web
6. check git --version
7. git clone https
8. select cloned file
9. Installing the SDK and Connecting to the Workspace:
    %pip install azure-ai-ml azure-identity scikit-learn joblib pandas -q
    from azure.ai.ml import MLClient
    from azure.identity import DefaultAzureCredential
get these 3 values from: studio -> top-right workspacename -> download config

    subscription_id = "d518a2fb-3458-4d55-bb95-54c32e1d9f0e"
    resource_group  = "lab9"
    workspace_name  = "lab9-kaustav-ws"

    ml_client = MLClient(
        DefaultAzureCredential(),
        subscription_id,
        resource_group,
        workspace_name,
   )

   print("Connected to workspace:", ml_client.workspace_name)

   10. register model in azure ml
       from azure.ai.ml.entities import Model
       from azure.ai.ml.constants import AssetTypes

       model_entity = Model(
            path="model.pkl",
             type = AssetTypes.CUSTOM_MODEL,
            name="iris-logreg-model",
             description="   "
       )
       registered_model = ml_client.models.create_or_update(model_entity)
       print(f"registered: {registered_model.name}, version{registered_model.version}")

   11. %%writefile score.py
       import json
       import os
       import joblib
       import numpy as np

       def init():
           global model
           model_path =os.path.join(os.getenv("AZUREML_MODEL_DIR"),"model.pkl")
           model = joblib.load(model_path)
       def run(raw_data):
            data = np.array(json.loads(raw_data)["data"])
             predictions = model.predict(data)
             return predictions.tolist()
   12. %%writefile conda.yaml
       name:
       channels:
          - defaults
       dependencies:
          - python=3.10
          - pip
          - pip:
              - scikit-learn
              - joblib
              - azureml-defaults
              - numpy
      13. create online endpoint
          from azure.ai.ml.entities import ManagedOnlineEndpoint
          import uuid

          endpoint = ManagedOnlineEndpoint(
                 name=endpoint_name,
                 description="endpoint",
                 auth_mode="key",
          )
          ml_client.online_endpoints.begin_create_or_update(endpoint_.result()
          print("endpoint created:", endpoint_name)

   14. deploy model to endpoint
       from azure.ai.ml.entities import ManagedOnlineDeployment, CodeConfiguration

       deployment = ManagedOnlineDeployment(
              name="blue",
              endpoint_name = endpoint_name,
              model=registered_model,
              environment=env,
              code_configuration=CodeConfiguration(
               code=".",
               scoring_script="score.py",
               ),
               instance-type = "Standard_DS2_v2",
                instance_count =1,
       )
       ml.client.online_deployments.begin_create_or_update(deployment).result()
       endpoint.traffic = {"blue":100)
       ml_client.online_endpoints.begin_create_or_update(endpoint).result()

       print("deployment complete")
       


    
