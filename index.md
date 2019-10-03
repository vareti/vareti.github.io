### Using Kubernetes client-go patch API

To demonstrate client-go patch API, I am picking the use case of adding a label to kubernetes nodes. In this example, JSON merge patching mechanism will be used.

    import (
        ...
        "github.com/client-go/kubernetes"
        corev1 "k8s.io/api/core/v1"
        "encoding/json"
        jsonpatch "gopkg.in/evanphx/json-patch.v4"
        "k8s.io/apimachinery/pkg/types"
        ...
    )

    ...

    const (
        labelKey   = "demo-key"
        labelValue = "demo-value"
    )

    ...

    func labelNode(clientset *kubernetes.Clientset, node *corev1.Node) {
        //create a copy of node structure
        _, oldData := json.Marshal(node)
        
        //add the labels and create a copy of modified node structure
        node.Labels[labelKey] = labelValue
        _, newData := json.Marshal(node)

        //creates a merge patch from old and new data
        patch, err := jsonpatch.CreateMergePatch(oldData, newData)
        if err != nil {
            return err
        }
        
        //tell api server to use JSON merge
        _, err = clientset.CoreV1().Nodes().Patch(node.Name, types.MergePatchType, patch)
        if err != nil {
            return err
        }
    }

