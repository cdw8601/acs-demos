# Generateing Templates with ACS Engine

The Azure Container Service Engine (acs-engine) generates ARM (Azure
Resource Manager) templates for Docker enabled clusters on Microsoft
Azure with your choice of DCOS, Kubernetes, or Swarm
orchestrators. The input to acs-engine is a cluster definition file
which describes the desired cluster, including orchestrator, features,
and agents. The structure of the input files is very similar to the
public API for Azure Container Service.

`acs-engine` reads a JSON cluster definiton and generates a number of
files that may be submitted to Azure Resource Manager (ARM). The
generated files include:

  1. **apimodel.json**: is an expanded version of the cluster
     definition provided to the generate command. All default or
     computed values will be expanded during the generate phase.
  2. **azuredeploy.json**: represents a complete description of all
     Azure resources required to fulfill the cluster definition from
     `apimodel.json`.
  3. **azuredeploy.parameters.json**: the parameters file holds a
     series of custom variables which are used in various locations
     throughout `azuredeploy.json`.
  4. **certificate and access config files**: orchestrators like
     Kubernetes require certificates and additional configuration
     files (e.g. Kubernetes apiserver certificates and kubeconfig).

## Generate templates from a cluster definition

As an example we will generate a template using a Kubernetes example
available in the [ACS Engine](http://github.com/azure/acs-engine)
GitHub repo.

### Edit the example cluster defininton 

We'll use
the
[Kubernetes](https://raw.githubusercontent.com/Azure/acs-engine/master/examples/kubernetes.json) example
cluster definition as a starter, lets copy this into our workspace:

```
wget -O $ACSE_WORKSPACE/kubernetes.json https://raw.githubusercontent.com/Azure/acs-engine/master/examples/kubernetes.json
```

Results:

```
--2017-07-26 17:29:05--  https://raw.githubusercontent.com/Azure/acs-engine/master/examples/kubernetes.json
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.52.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.52.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 740 [text/plain]
Saving to: ‘~/.acs-engine/simdem/kuberenetes.json’

~/.acs-engine/simde 100%[===================>]     740  --.-KB/s    in 0s

2017-07-26 17:29:05 (214 MB/s) - ‘~/.acs-engine/simdem/kuberenetes.json’ saved [740/740]
```

```
cat $ACSE_WORKSPACE/kubernetes.json
```

Results:

```
{ "apiVersion": "vlabs", 
  "properties": { "orchestratorProfile": {
    "orchestratorType": "Kubernetes" 
  }, 
  "masterProfile": { 
    "count": 1,
    "dnsPrefix": "", 
	"vmSize": "Standard_D2_v2" 
  }, 
  "agentPoolProfiles": [ 
    { 
	  "name": "agentpool1", 
	  "count": 3, 
	  "vmSize": "Standard_D2_v2",
	  "availabilityProfile": "AvailabilitySet" 
	} ],
	"linuxProfile": {
	  "adminUsername": "azureuser", 
	  "ssh": { 
	    "publicKeys": [ 
		  { "keyData": "" } 
	    ] 
	  } 
	}, 
	"servicePrincipalProfile": { 
	  "servicePrincipalClientID": "", 
	  "servicePrincipalClientSecret": "" 
	} 
  } 
}
```

In this file we must, at a minimum, provide values for:

  1. dnsPrefix
  2. keyData
  3. servicePrincipalClientID
  4. servicePrincipalClientSecret

Setting dnsPrefix

```
tmp_file=$(mktemp)
```

```
jq ".properties.masterProfile.dnsPrefix |= \"$ACSE_DNS_PREFIX\"" $ACSE_WORKSPACE/kubernetes.json  > "$tmp_file"
mv $tmp_file $ACSE_WORKSPACE/kubernetes.json
```
Setting ssh keyData

```
public_key=$(ssh-keygen -y -f $ACSE_SSH_KEY)
tmp_file=$(mktemp)
```

```
jq ".properties.linuxProfile.ssh.publicKeys[0].keyData |= \"$public_key\"" $ACSE_WORKSPACE/kubernetes.json  > "$tmp_file"
mv $tmp_file $ACSE_WORKSPACE/kubernetes.json
```

Setting the Service Principle.

```
client_id=$(az ad sp show --id http://acs-engine-simdem --query appId --output tsv)
tmp_file=$(mktemp)
```

```
jq ".properties.servicePrincipalProfile.servicePrincipalClientID |= \"$client_id\"" $ACSE_WORKSPACE/kubernetes.json  > "$tmp_file"
mv $tmp_file $ACSE_WORKSPACE/kubernetes.json
```

Now we need to set the client secret. This was captured in
`$client_sceret` when the service principle was created earlier.

```
tmp_file=$(mktemp)
```

```
jq ".properties.servicePrincipalProfile.servicePrincipalClientSecret |= \"$ACSE_SERVICE_PRINCIPLE_PASSWORD\"" $ACSE_WORKSPACE/kubernetes.json  > "$tmp_file"
mv $tmp_file $ACSE_WORKSPACE/kubernetes.json
```

The complete config file is now:

```
cat $ACSE_WORKSPACE/kubernetes.json
```


### Generate the templates

```
acs-engine generate $ACSE_WORKSPACE/kubernetes.json --output-directory $ACSE_WORKSPACE/_output
```

Results:

```
INFO[0000] Generating assets...
```

The assets should now be available in the _output directory:

```
ls $ACSE_WORKSPACE/_output/
```

Results:

```expected_similarity=0.9
apimodel.json  azuredeploy.json             ca.key      kubeconfig
apiserver.crt  azuredeploy.parameters.json  client.crt  kubectlClient.crt
apiserver.key  ca.crt                       client.key  kubectlClient.key
```

# Next Steps

  1. [Deploy ACS Engine Templates with Azure CLI](../deploy/script,md)
  2. [Cleanup ACS Engine SimdDem](../cleanup/script.md)
