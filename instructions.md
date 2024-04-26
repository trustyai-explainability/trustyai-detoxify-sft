## Instructions to run end-to-end demo

## Chapters
[I. Installation of KServe & its dependencies](#installation-of-kserve--its-dependencies)

[II. Setting up local MinIO S3 storage](#setting-up-local-minio-s3-storage)

[III. Setting up your OpenShift AI workbench](#setting-up-your-openshift-ai-workbench)

[IV. Train model and evaluate](#train-model-and-evaluate)

[V. Convert model to Caikit format and save to S3 storage](#convert-model-to-caikit-format-and-save-to-s3-storage)

[V. Deploy model onto Caikit-TGIS Serving Runtime](#deploy-model-onto-caikit-tgis-serving-runtime)

[VI. Model inference](#model-inference)

**Prerequisites**
* To support training and inference, your cluster needs a node with CPUS, 4 GPUs, and GB memory. Instructions to add GPU support to RHOAI can be found [here](https://docs.google.com/document/d/1T2oc-KZRMboUVuUSGDZnt3VRZ5s885aDRJGYGMkn_Wo/edit#heading=h.9xmhoufikqid).
* You have a cluster administrator permissions
* You have installed the OpenShift CLI (`oc`)
* You have installed the `Red Hat OpenShift Service Mesh Operator`
* You have installed the `Red Hat OpenShift Serverless Operator`
* You have installed the `Red Hat OpenShift AI Operator` and created a **DataScienceCluster** object


### Installation of KServe & its dependencies
Instructions adapted from [Manually installing KServe](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai_self-managed/2-latest/html/serving_models/serving-large-models_serving-large-models#manually-installing-kserve_serving-large-models)
1. Git clone this repository
    ```
    git clone https://github.com/trustyai-explainability/trustyai-detoxify-sft.git
    ```

2. Login to your OpenShift cluster as a cluster adminstrator
    ```
    oc login --token=<token>
    ```
2. Create the required namespace for Red Hat OpenShift Service Mesh
    ```
    oc create ns istio-system
    ```

3. Create a `ServiceMeshControlPlane` object
    ```
    oc apply -f manifests/kserve/smcp.yaml -n istio-system
    ```
4. Sanity check to verify creation of the service mesh instance
    ```
    oc get pods -n istio-system
    ```
    Expected output:
    ```
    NAME                                          READY   STATUS   	  RESTARTS    AGE
    istio-egressgateway-7c46668687-fzsqj          1/1     Running     0           22h
    istio-ingressgateway-77f94d8f85-fhsp9         1/1     Running     0           22h
    istiod-data-science-smcp-cc8cfd9b8-2rkg4      1/1     Running     0           22h
    ```

5. Create the required namespace for a `KnativeServing` instance
    ```
    oc create ns knative-serving
    ```

6. Create a `ServiceMeshMember` object
    ```
    oc apply -f manifests/kserve/default-smm.yaml -n knative-serving
    ```

7. Create and define a `KnativeServing` object
    ```
    oc apply -f manifests/kserve/knativeserving-istio.yaml -n knative-serving
    ```
8. Sanity check to validate creation of the Knative Serving instance
    ```
    oc get pods -n knative-serving
    ```
    Expected output:
    ```
    NAME                                     	READY       STATUS    	RESTARTS   	AGE
    activator-7586f6f744-nvdlb               	2/2         Running   	0          	22h
    activator-7586f6f744-sd77w               	2/2         Running   	0          	22h
    autoscaler-764fdf5d45-p2v98             	2/2         Running   	0          	22h
    autoscaler-764fdf5d45-x7dc6              	2/2         Running   	0          	22h
    autoscaler-hpa-7c7c4cd96d-2lkzg          	1/1         Running   	0          	22h
    autoscaler-hpa-7c7c4cd96d-gks9j         	1/1         Running   	0          	22h
    controller-5fdfc9567c-6cj9d              	1/1         Running   	0          	22h
    controller-5fdfc9567c-bf5x7              	1/1         Running   	0          	22h
    domain-mapping-56ccd85968-2hjvp          	1/1         Running   	0          	22h
    domain-mapping-56ccd85968-lg6mw          	1/1         Running   	0          	22h
    domainmapping-webhook-769b88695c-gp2hk   	1/1         Running     0          	22h
    domainmapping-webhook-769b88695c-npn8g   	1/1         Running   	0          	22h
    net-istio-controller-7dfc6f668c-jb4xk    	1/1         Running   	0          	22h
    net-istio-controller-7dfc6f668c-jxs5p    	1/1         Running   	0          	22h
    net-istio-webhook-66d8f75d6f-bgd5r       	1/1         Running   	0          	22h
    net-istio-webhook-66d8f75d6f-hld75      	1/1         Running   	0          	22h
    webhook-7d49878bc4-8xjbr                 	1/1         Running   	0          	22h
    webhook-7d49878bc4-s4xx4                 	1/1         Running   	0          	22h
    ```

9. From the web console, install KServe by going to **Operators -> Installed Operators** and click on the **Red Hat OpenShift AI Operator**

10. Click on the **DSC Intialization** tab and click on the **default-dsci** object

11. Click on the **YAML** tab and in the `spec` section, change the `serviceMesh.managementState` to `Unmanaged`
    ```
    spec:
    serviceMesh:
    managementState: Unmanaged
    ```

12. Click **Save**

12. Click on the **Data Science Cluster** tab and click on the **default-dsc** object

13. Click on the **YAML** tab and in the `spec` section, change the `components.kserve.managementState` and the `components.kserve.serving.managementState` to `Managed`
    ```
    spec:
    components:
    kserve:
        managementState: Managed
        serving:
            managementState: Managed

    ```
15. Click **Save**

### Setting up local MinIO S3 storage
1. Create a namespace for your project called "detoxify-sft"
    ```
    oc create namespace detoxify-sft
    ```
2. Set up your local MinIO S3 storage in your newly created namespace
    ```
    oc apply -f manifests/minio/setup-s3.yaml -n detoxify-sft
    ```
3. Run the following sanity checks
    ```
    oc get pods -n detoxify-sft | grep "minio"
    ```
    Expected output:
    ```
    NAME                                     	READY       STATUS    	RESTARTS   	AGE
    minio-7586f6f744-nvdl                       1/1         Running     0           22h
    ```

    ```
    oc get route -n detoxify-sft | grep "minio"
    ```
    Expected output:
    ```
    NAME                                        STATUS    	LOCATION   	            SERVICE
    minio-api                                   Accepted    https://minio-api...    minio-service
    minio-ui                                    Accepted    https://minio-ui...     minio-service
    ```
4.  Get the MinIO UI location URL and open it in a web browser
    ```
    oc get route minio-ui -n detoxify-sft
    ```
5. Login using the credentials in `manifests/minio/setup-s3.yaml`

    **user**: `minio`

    **password**: `minio123`

6. Click on **Create a Bucket** and choose a name for your bucket and click on **Create Bucket**

### Setting up your OpenShift AI workbench
1. Go to Red Hat OpenShift AI from the web console

2. Click on **Data Science Projects** and then click on **Create data science project**

3. Give your project a name and then click **Create**

4. Click on the **Workbenches** tab and then create a workbench with a Pytorch notebook image, set the container size to Large, and select a single NVIDIA GPU. Click on **Create Workbench**

5. Click on **Add data connection** to create a matching data connection for MinIO

6. Fill out the required fields and then click on **Add data collection**

7. Once your workbench status changes from **Starting** to **Running**, click on **Open** to open JupyterHub in a web browser

8. In your JupyterHub environment, launch a terminal and clone this project
    ```
    git clone https://github.com/trustyai-explainability/trustyai-detoxify-sft.git
    ```
8. Go into the `notebooks` directory

### Train model and evaluate
1.  Open the `01-sft.ipynb` file

2. Run each cell in the notebook

3. Once the model trained and uploaded to HuggingFace Hub, open the `02-eval.ipynb` file and run each cell to compare the model trained on raw input-output pairs vs. the one trained on detoxified prompts

### Convert model to Caikit format and save to S3 storage
1. Open the `03-save_convert_model.ipynb` and run each cell in the notebook to convert the model Caikit format and save it to a MinIO bucket

### Deploy model onto Caikit-TGIS Serving Runtime
1. In the OpenShift AI dashboard, navigate to the  project details page and click the **Models** tab

2. In the **Single-model serving platform** tile, click on deploy model. Provide the following values:

   **Model Name**: `opt-350m-caikit`

   **Serving Runtime**: `Caikit-TGIS Serving Runtime`

   **Model framework**: `caikit`

   **Existing data connection**: `My Storage`

   **Path**: `models/opt-350m-caikit`

3. Click **Deploy**

4. Increase the `initialDelaySeconds`
    ```
    oc patch template caikit-tgis-serving-template  --type=='merge' -p '{"spec":{"containers":[{"readinessProbe":"initialDelaySeconds":300, "livenessProbe":"initialDelaySeconds":300}]}}'
    ```
5. Wait for the model **Status** to show a green checkmark

### Model inference
1. Return to the JupyterHub environment to test out the deployed model

2. Click on `03-inference_request.ipynb` and run each cell to make an inference request to the detoxified model
