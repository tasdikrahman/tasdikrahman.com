---
layout: post
title: "Musings with client-go of k8s"
description: "Musings with client-go of k8s"
tags: [kubernetes, golang]
comments: true
share: true
cover_image: ''
---

This post mostly is for documentary purposes for myself, about a few things which I ended up noticing while using [client-go](https://github.com/kubernetes/client-go) as I used it for [deliveryhero/k8s-cluster-upgrade-tool](https://github.com/deliveryhero/k8s-cluster-upgrade-tool), which used the out-cluster client configuration, a couple of things are specific to that setup, like client init, but other things like testing interactions via client-go are more generic.

## Initialization of the config

client-go in itself, shows a couple of example of client init [here](https://github.com/kubernetes/client-go/blob/release-1.21/examples/out-of-cluster-client-configuration/main.go#L44-L62), pasting the snippet here for context

```golang
...
var kubeconfig *string
if home := homedir.HomeDir(); home != "" {
  kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
} else {
	kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
}
flag.Parse()

// use the current context in kubeconfig
config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
if err != nil {
	panic(err.Error())
}

// create the clientset
clientset, err := kubernetes.NewForConfig(config)
if err != nil {
	panic(err.Error())
}
...
```

This would by default go ahead and initialize the client with the current k8s context you are attached to. For example in your `~/.kube/config` file

```sh
...
current-context: foo-cluster
kind: Config
preferences: {}
...
```

with the above, the client will get initliazed with the k8s context being `foo-cluster`.

### Initiliazation happening by default to the default kube context

What ends up happening underneath is that, the method [`BuildConfigFromFlags()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L613), which goes on to call [`ClientConfig()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L134), which is where the k8s context is deduced, when the call to [`getContext()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L144) gets made. The flow defaults to the current context via the call made to [`getContextName()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L462).

This is also where we notice that [we can add an override](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L427), which if added, would select that particular context, instead of choosing the default, which is picked up from the current context lready selected in the `~/.kube/context`.

So how do we go about overriding this?

### Initializing client-go to a user specified k8s context

We would just need to have the [`Context`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/overrides.go#L35) added to [`ConfigOverrides`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/overrides.go#L30), when the call to ClientConfig() gets made at the in [`BuildConfigFromFlags()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L613)

```go
// buildConfigFromFlags returns the config using which the client will be initialized with the k8s context we want to use
func buildConfigFromFlags(context, kubeconfigPath string) (*rest.Config, error) {
	return clientcmd.NewNonInteractiveDeferredLoadingClientConfig(
		&clientcmd.ClientConfigLoadingRules{ExplicitPath: kubeconfigPath},
		&clientcmd.ConfigOverrides{
			CurrentContext: context,
		}).ClientConfig()
}

// KubeClientInit returns back clientSet
func KubeClientInit(kubeContext string) (*kubernetes.Clientset, error) {
	var kubeConfig *string
	if home := homedir.HomeDir(); home != "" {
		kubeConfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		kubeConfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}
	flag.Parse()

	config, err := buildConfigFromFlags(kubeContext, *kubeConfig)
	if err != nil {
		return &kubernetes.Clientset{}, errors.New("error building the config for building the client-set for client-go")
	}

	// create the clientset
	clientSet, err := kubernetes.NewForConfig(config)
	if err != nil {
		return &kubernetes.Clientset{}, errors.New("error building the client-set for client-go")
	}
	return clientSet, nil
}
```

the init would then look like

```go
...
kubeClient, err := k8s.KubeClientInit("cluster-name")
...
```

## Testing client-go interactions

While writing a couple of interactions on [deliveryhero/k8s-cluster-upgrade-tool](https://github.com/deliveryhero/k8s-cluster-upgrade-tool), ended up checking what were folks doing to add specs for interactions with client-go, and we already had the [https://pkg.go.dev/k8s.io/client-go/kubernetes/fake](https://pkg.go.dev/k8s.io/client-go/kubernetes/fake) package out there which one could use to test the interactions with the client.

My specific cases for testing were quite simple to setup. The first thing again, which you would need to take care of is obviously, that the method which you want to test, should be able to take the client as a dependency which you inject in the spec. After that's it's super simple.

For example, for one case, I would want a behaviour, where querying a specific object type, when queried with specific attributes, would return me back that object. Then the methods behaviour of how it processed that output would be put under test.

```go
...
client := fake.NewSimpleClientset(&appsv1.DaemonSet{
					ObjectMeta: metav1.ObjectMeta{
						Name:      "aws-node",
						Namespace: "kube-system",
					},
					Spec: appsv1.DaemonSetSpec{
						Template: corev1.PodTemplateSpec{
							Spec: corev1.PodSpec{
								Containers: []corev1.Container{
									{
										Image: "aws-node:v1.0.0",
									},
								},
							},
						},
					},
				})
...
```

Now any interactions which we would want to have with the client, where if we query for this object via the specific api's we would get this object back.

```go
func myMethod(k8sClient kubernetes.Interface, myObjects...) {
  // when the subject is under test, this method when passed the fake client with the above initialization would have the aws-node object present
  // which would be returned.
  daemonSet, err := k8sClient.AppsV1().DaemonSets(namespace).Get(context.TODO(), k8sObjectName, metav1.GetOptions{})
  ...
  ...
}
```

To give a more full example, here is a case where function returns the container image of the first container for a deployment of a daemonset object

```go
func GetContainerImageForK8sObject(k8sClient kubernetes.Interface, k8sObjectName, k8sObject, namespace string) (string, error) {
	switch k8sObject {
	case "deployment":
		// NOTE: Not targeting other api versions for the objects as of now.
		deployment, err := k8sClient.AppsV1().Deployments(namespace).Get(context.TODO(), k8sObjectName, metav1.GetOptions{})
		if k8sErrors.IsNotFound(err) {
			return "", fmt.Errorf("Deployment %s in namespace %s not found\n", k8sObjectName, namespace)
		} else if statusError, isStatus := err.(*k8sErrors.StatusError); isStatus {
			return "", fmt.Errorf("Error getting deployment %s in namespace %s: %v\n",
				k8sObjectName, namespace, statusError.ErrStatus.Message)
		} else if err != nil {
			return "", fmt.Errorf("there was an error while retrieving the container image")
		}

		// NOTE: This assumes there is only one container in the k8s object, which is true for the components for us at moment
		return deployment.Spec.Template.Spec.Containers[0].Image, nil
	case "daemonset":
		// NOTE: Not targeting other api versions for the objects as of now.
		daemonSet, err := k8sClient.AppsV1().DaemonSets(namespace).Get(context.TODO(), k8sObjectName, metav1.GetOptions{})
		if k8sErrors.IsNotFound(err) {
			return "", fmt.Errorf("daemonset %s in namespace %s not found\n", k8sObjectName, namespace)
		} else if statusError, isStatus := err.(*k8sErrors.StatusError); isStatus {
			return "", fmt.Errorf(fmt.Sprintf("Error getting daemonset %s in namespace %s: %v\n",
				k8sObjectName, namespace, statusError.ErrStatus.Message))
		} else if err != nil {
			return "", fmt.Errorf("there was an error while retrieving the container image")
		}

		// NOTE: This assumes there is only one container in the k8s object, which is true for the components for us at moment
		return daemonSet.Spec.Template.Spec.Containers[0].Image, nil
	default:
		return "", fmt.Errorf("please choose between Daemonset or Deployment k8sobject as they are currently supported")
	}
}
```

Specific case when the object is of kind `Deployment`

```go
func TestGetContainerImageForK8sObjectWhenK8sObjectIsDeployment(t *testing.T) {
	type deploymentArgs struct {
		k8sObject     string
		k8sObjectName string
		kubeContext   string
		namespace     string
		deployment    *appsv1.Deployment
	}
	tests := []struct {
		name   string
		args   deploymentArgs
		err    error
		output string
	}{
		{
			name: "When the object is of type deployment, the objectname is cluster-autoscaler, object exists and returns back the image",
			args: deploymentArgs{k8sObject: "deployment", k8sObjectName: "cluster-autoscaler", kubeContext: "test-context", namespace: "kube-system",
				deployment: &appsv1.Deployment{
					ObjectMeta: metav1.ObjectMeta{
						Name:      "cluster-autoscaler",
						Namespace: "kube-system",
					},
					Spec: appsv1.DeploymentSpec{
						Template: corev1.PodTemplateSpec{
							Spec: corev1.PodSpec{
								Containers: []corev1.Container{
									{
										Image: "cluster-autoscaler:v1.0.0",
									},
								},
							},
						},
					},
				}},
			output: "cluster-autoscaler:v1.0.0",
			err:    nil,
		},
		{
			name: "When the object is of type deployment, the objectname is cluster-autoscaler, object doesn't exist, returns back error",
			args: deploymentArgs{k8sObject: "deployment", k8sObjectName: "cluster-autoscaler", kubeContext: "test-context", namespace: "kube-system",
				deployment: &appsv1.Deployment{}},
			output: "",
			err:    errors.New("Deployment cluster-autoscaler in namespace kube-system not found\n"),
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			client := fake.NewSimpleClientset(tt.args.deployment)

			got, err := GetContainerImageForK8sObject(client, tt.args.k8sObjectName, tt.args.k8sObject, tt.args.namespace)

			assert.Equal(t, tt.output, got)
			assert.Equal(t, tt.err, err)
		})
	}
}
```

A full example of the same which I wrote is here where [`GetContainerImageForK8sObject()`](https://github.com/deliveryhero/k8s-cluster-upgrade-tool/blob/d19b863c9daec01e3f5378ae7a32fb3bf94e86bd/internal/api/k8s/cluster.go#L111) is the function under test, which takes the client as a dependency and then in the test case we would use the fake client [here like this](https://github.com/deliveryhero/k8s-cluster-upgrade-tool/blob/v0.4.0/internal/api/k8s/cluster_test.go#L44-L100)


## References

- [https://github.com/kubernetes/client-go](https://github.com/kubernetes/client-go)
- [https://github.com/deliveryhero/k8s-cluster-upgrade-tool](https://github.com/deliveryhero/k8s-cluster-upgrade-tool)
- [https://pkg.go.dev/k8s.io/client-go/kubernetes/fake](https://pkg.go.dev/k8s.io/client-go/kubernetes/fake)
- [`BuildConfigFromFlags()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L613)
- [`ClientConfig()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L134)
- [`getContextName()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L462)
- [`getContext()`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/client_config.go#L144)
- [`ConfigOverrides`](https://github.com/kubernetes/client-go/blob/1110612dc6e599ae817abbcb762c7c5e87e99a51/tools/clientcmd/overrides.go#L30)
