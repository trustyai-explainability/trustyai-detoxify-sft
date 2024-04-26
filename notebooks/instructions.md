## Instructions to run end-to-end demo

## Chapters

**Prerequisites**
* To support training and inference, your cluster need a node with CPUS, 4 GPUs, and GB memory
* You have a cluster administrator permissions
* You have installed the OpenShift CLI (`oc`)
* You have installed the `Red Hat OpenShift Service Mesh Operator`
* You have installed the `Red Hat OpenShift Serverless Operator`
* You have installed the `Red Hat OpenShift AI Operator` and created a **DataScienceCluster** object


### Installation of KServe & dependencies
Instructions adapted from [Manually installing KServe](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai_self-managed/2-latest/html/serving_models/serving-large-models_serving-large-models#manually-installing-kserve_serving-large-models)
1. Git clone this repository
    ```
    git clone
    ```

2. Login to your OpenShift cluster as a cluster adminstrator
    ```
    oc login <token>
    ```
2. Create the required namespace for Red Hat OpenShift Service Mesh
    ```
    oc create ns istio-system
    ```

3. Create a `ServiceMeshControlPlane` object
    ```
    oc apply -f ../manifests/kserve/smcp.yaml -n istio-system
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
    oc apply -f ../manifests/kserve/default-smm.yaml -n knative-serving
    ```

7. Create and define a `KnativeServing` object
    ```
    oc apply -f ../manifests/kserve/knativeserving-istio.yaml -n knative-serving
    ```
8. Sanity check to validate creation of the Knative Serving instance
    ```
    oc get pods -n knative-serving
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
            managementState: Unmanaged

    ```
15. Click **Save**

### Setting up your OpenShift AI workbench
1. Go to Red Hat OpenShift AI from the web console


2. Click on **Data Science Projects** and then click on **Create data science project**

3. Give your project a name and then click **Create**


4. Create a workbench with a Pytorch notebook image and set the container size to Large
