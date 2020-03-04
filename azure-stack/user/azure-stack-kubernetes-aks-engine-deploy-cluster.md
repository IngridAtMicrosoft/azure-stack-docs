---
title: Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub 
description: How to deploy a Kubernetes cluster on Azure Stack Hub from a client VM running the AKS engine. 
author: mattbriggs

ms.topic: article
ms.date: 01/10/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 11/21/2019

# Intent: Notdone: As a < type of user >, I want < what? > so that < why? >
# Keyword: Notdone: keyword noun phrase

---


# Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub

You can deploy a Kubernetes cluster on Azure Stack Hub from a client VM running the AKS engine. In this article, we look at writing a cluster specification, deploying a cluster with the `apimodel.json` file, and checking your cluster by deploying MySQL with Helm.

## Define a cluster specification

You can specify a cluster specification in a document file using the JSON format called the [API model](https://github.com/Azure/aks-engine/blob/master/docs/topics/architecture.md#architecture-diagram). The AKS engine uses a cluster specification in the API model to create your cluster. 

### Update the API model

This section looks at creating an API model for your cluster.

1.  Start by using an Azure Stack Hub [example](https://github.com/Azure/aks-engine/tree/master/examples/azure-stack) API Model file and make a local copy for your deployment. From the machine, you installed AKS engine, run:

    ```bash
    curl -o kubernetes-azurestack.json https://raw.githubusercontent.com/Azure/aks-engine/master/examples/azure-stack/kubernetes-azurestack.json
    ```

    > [!Note]  
    > If you are disconnected, you can download the file and manually copy it to the disconnected machine where you plan to edit it. You can copy the file to your Linux machine using tools such as [PuTTY or WinSCP](https://www.suse.com/documentation/opensuse103/opensuse103_startup/data/sec_filetrans_winssh.html).

2.  To open the  in an editor, you can use nano:

    ```bash
    nano ./kubernetes-azurestack.json
    ```

    > [!Note]  
    > If you don't have nano installed, you can install nano on Ubuntu: `sudo apt-get install nano`.

3.  In the kubernetes-azurestack.json file, find `orchestratorRelease`. Select one of the supported Kubernetes versions. For example, 1.14, 1.15. The versions are often updates. Specify the version as x.xx rather than x.xx.x. For a list of current versions, see [Supported Kubernetes Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-kubernetes-versions). You can find out the supported version by running the following AKS engine command:

    ```bash
    aks-engine get-versions
    ```

4.  Find `customCloudProfile` and provide the URL to the tenant portal. For example, `https://portal.local.azurestack.external`. 

5. Add `"identitySystem":"adfs"` if you're using AD FS. For example,

    ```JSON  
        "customCloudProfile": {
            "portalURL": "https://portal.local.azurestack.external",
            "identitySystem": "adfs"
        },
    ```

    > [!Note]  
    > If you're using Azure AD for your identity system, you don't need to add the **identitySystem** field.

6. Find `portalURL` and provide the URL to the tenant portal. For example, `https://portal.local.azurestack.external`.

7.  In the array `masterProfile`, set the following fields:

    | Field | Description |
    | --- | --- |
    | dnsPrefix | Enter a unique string that will serve to identify the hostname of VMs. For example, a name based on the resource group name. |
    | count |  Enter the number of masters you want for your deployment. The minimum for an HA deployment is 3, but 1 is allowed for non-HA deployments. |
    | vmSize |  Enter [a size supported by Azure Stack Hub](https://docs.microsoft.com/azure-stack/user/azure-stack-vm-sizes), example `Standard_D2_v2`. |
    | distro | Enter `aks-ubuntu-16.04`. |

8.  In the array `agentPoolProfiles` update:

    | Field | Description |
    | --- | --- |
    | count | Enter the number of agents you want for your deployment. |
    | vmSize | Enter [a size supported by Azure Stack Hub](https://docs.microsoft.com/azure-stack/user/azure-stack-vm-sizes), example `Standard_D2_v2`. |
    | distro | Enter `aks-ubuntu-16.04`. |

9.  In the array `linuxProfile` update:

    | Field | Description |
    | --- | --- |
    | adminUsername | Enter the VM admin user name. |
    | ssh | Enter the public key that will be used for SSH authentication with VMs. If you are using Putty, open PuTTY Key Generator to load the Putty private key and the public key which starts with ssh-rsa as in the following example. You can use the key generated when creating the Linux client but **you need to copy the public key so that it is a single line text as shown in the example**.|

    ![PuTTY key generator](media/azure-stack-kubernetes-aks-engine-deploy-cluster/putty-key-generator.png)

### More information about the API model

- For a complete reference of all the available options in the API model, refer to the [Cluster definitions](https://github.com/Azure/aks-engine/blob/master/docs/topics/clusterdefinitions.md).  
- For highlights on specific options for Azure Stack Hub, refer to the [Azure Stack Hub cluster definition specifics](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#cluster-definition-aka-api-model).  

## Deploy a Kubernetes cluster

After you have collected all the required values in your API model, you can create your cluster. At this point you should:

Ask your Azure Stack Hub operator to:

- Verify the health of the system, suggest running `Test-AzureStack` and your OEM vendor's hardware monitoring tool.
- Verify the system capacity including resources such as memory, storage, and public IPs.
- Provide details of the quota associated with your subscription so that you can verify that there is still enough space for the number of VMs you plan to use.

Proceed to deploy a cluster:

1.  Review the available parameters for AKS engine on Azure Stack Hub [CLI flags](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#cli-flags).

    | Parameter | Example | Description |
    | --- | --- | --- |
    | azure-env | AzureStackCloud | To indicate to AKS engine that your target platform is Azure Stack Hub use `AzureStackCloud`. |
    | identity-system | adfs | Optional. Specify your identity management solution if you are using Active Directory Federated Services (AD FS). |
    | location | local | The region name for your Azure Stack Hub. For the ASDK the region is set to `local`. |
    | resource-group | kube-rg | Enter the name of a new resource group or select an existing resource group. The resource name needs to be alphanumeric and lowercase. |
    | api-model | ./kubernetes-azurestack.json | Path to the cluster configuration file, or API model. |
    | output-directory | kube-rg | Enter the name of the directory to contain the output file `apimodel.json` as well as other generated files. |
    | client-id | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Enter the service principal GUID. The Client ID identified as the Application ID when your Azure Stack Hub administrator created the service principal. |
    | client-secret | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Enter the service principal secret. This is the client secret you set up when creating your service. |
    | subscription-id | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Enter your Subscription ID. For more information see [Subscribe to an offer](https://docs.microsoft.com/azure-stack/user/azure-stack-subscribe-services#subscribe-to-an-offer) |

    Here is an example:

    ```bash  
    aks-engine deploy \
    --azure-env AzureStackCloud \
    --location <for asdk is local> \
    --resource-group kube-rg \
    --api-model ./kubernetes-azurestack.json \
    --output-directory kube-rg \
    --client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --client-secret xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --identity-system adfs # required if using AD FS
    ```

2.  If for some reason the execution fails after the output directory has been created, you can correct the issue and rerun the command. If you are rerunning the deployment and had used the same output directory before, the AKS engine will return an error saying that the directory already exists. You can overwrite the existing directory by using the flag: `--force-overwrite`.

3.  Save the AKS engine cluster configuration in a secure, encrypted location.

    Locate the file `apimodel.json`. Save it to a secure location. This file will be used as input in all of your other AKS engine operations.

    The generated `apimodel.json` contains the service principal, secret, and SSH public key you use in the input API model. It also has all the other metadata needed by the AKS engine to perform all other operations. If you lose it, the AKS engine won't be able configure the cluster.

    The secrets are **unencrypted**. Keep the file in an encrypted, secure place. 

## Verify your cluster

Verify your cluster by deploying MySql with Helm to check your cluster.

1. Get the public IP address of one of your master nodes using the Azure Stack Hub portal.

2. From a machine with access to your Azure Stack Hub instance, connect via SSH into the new master node using a client such as PuTTY or MobaXterm. 

3. For the SSH username, you use "azureuser" and the private key file of the key pair you provided for the deployment of the cluster.

4. Run the following commands to create a sample deployment of a Redis master (for connected stamps only):

   ```bash
   kubectl apply -f https://k8s.io/examples/application/guestbook/redis-master-deployment.yaml
   ```

    1. Query the list of pods:

       ```bash
       kubectl get pods
       ```

    2. The response should be similar to the following:

       ```shell
       NAME                            READY     STATUS    RESTARTS   AGE
       redis-master-1068406935-3lswp   1/1       Running   0          28s
       ```

    3. View the deployment logs:

       ```shell
       kubectl logs -f <pod name>
       ```

    For a complete deployment of a sample PHP app that includes the Redis master, follow [the instructions here](https://kubernetes.io/docs/tutorials/stateless-application/guestbook/).

5. For a disconnected stamp, the following commands should be sufficient:

    1. First check that the cluster endpoints are running:

       ```bash
       kubectl cluster-info
       ```

       The output should look similar to the following:

       ```shell
       Kubernetes master is running at https://democluster01.location.domain.com
       CoreDNS is running at https://democluster01.location.domain.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
       kubernetes-dashboard is running at https://democluster01.location.domain.com/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
       Metrics-server is running at https://democluster01.location.domain.com/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
       ```

    2. Then, review node states:

       ```bash
       kubectl get nodes
       ```

       The output should be similar to the following:

       ```shell
       k8s-linuxpool-29969128-0   Ready      agent    9d    v1.15.5
       k8s-linuxpool-29969128-1   Ready      agent    9d    v1.15.5
       k8s-linuxpool-29969128-2   Ready      agent    9d    v1.15.5
       k8s-master-29969128-0      Ready      master   9d    v1.15.5
       k8s-master-29969128-1      Ready      master   9d    v1.15.5
       k8s-master-29969128-2      Ready      master   9d    v1.15.5
       ```

6. To clean up the redis POD deployment from the previous step, run the following command:

    ```bash
    kubectl delete deployment -l app=redis
    ```

## Next steps

> [!div class="nextstepaction"]
> [Troubleshoot the AKS engine on Azure Stack Hub](azure-stack-kubernetes-aks-engine-troubleshoot.md)
