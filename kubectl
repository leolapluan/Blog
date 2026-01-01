# Essential Kubectl Commands for Developers

Here is a cheat sheet of the most common `kubectl` commands you will use daily.

## 1. Inspecting Resources
Get a high-level view of your cluster.

*   **List all pods in the current namespace:**
    ```bash
    kubectl get pods
    ```
*   **List pods with extra details (IP, Node):**
    ```bash
    kubectl get pods -o wide
    ```
*   **Get detailed info about a specific resource:**
    ```bash
    kubectl describe pod <pod-name>
    ```

## 2. Debugging
Troubleshoot issues directly from the terminal.

*   **View logs for a specific pod:**
    ```bash
    kubectl logs <pod-name>
    ```
*   **Stream logs in real-time (like `tail -f`):**
    ```bash
    kubectl logs -f <pod-name>
    ```
*   **Execute a command inside a running container:**
    ```bash
    kubectl exec -it <pod-name> -- /bin/sh
    ```

## 3. Managing State
Apply changes or create resources relative to YAML files.

*   **Apply a configuration:**
    ```bash
    kubectl apply -f ./my-manifest.yaml
    ```
*   **Delete resources defined in a file:**
    ```bash
    kubectl delete -f ./my-manifest.yaml
    ```

## 4. Context & Configuration
Switch between clusters (e.g., dev, prod) easily.

*   **Check current context:**
    ```bash
    kubectl config current-context
    ```
*   **Switch to a different namespace:**
    ```bash
    kubectl config set-context --current --namespace=<namespace>
    ```

## 5. Common Problems & Debugging
When things go wrong, use these patterns to find the root cause.

### 🔴 Status: `CrashLoopBackOff`
The container starts but immediately crashes, repeatedly.
*   **Suspect**: Application panic, missing env variable, or configuration error.
*   **Debug**: Check the logs of the *previous* crashed instance.
    ```bash
    kubectl logs <pod-name> --previous
    ```

### 🔴 Status: `ImagePullBackOff` / `ErrImagePull`
Kubernetes cannot download your container image.
*   **Suspect**: Typo in image name/tag, private registry authentication missing, or registry is down.
*   **Debug**: Describe the pod and look at the "Events" section.
    ```bash
    kubectl describe pod <pod-name>
    # Search for "Failed to pull image" in Events
    ```

### 🔴 Status: `Pending`
The pod is created but not running on any node.
*   **Suspect**: Cluster is out of CPU/Memory, or no node matches the required labels/taints.
*   **Debug**: The Scheduler will tell you exactly why in the events.
    ```bash
    kubectl describe pod <pod-name>
    # Look for "Insufficient cpu" or "MatchNodeSelector" in Events
    ```
