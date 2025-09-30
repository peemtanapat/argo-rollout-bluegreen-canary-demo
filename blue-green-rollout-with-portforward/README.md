a step-by-step guide to set up blue-green deployment with Argo Rollouts on your MacOS with Docker Desktop Kubernetes:


## Step 1: Install Argo Rollouts on Kubernetes

```bash
# Create namespace for Argo Rollouts
kubectl create namespace argo-rollouts

# Install Argo Rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Verify installation
kubectl get pods -n argo-rollouts
```

## Step 2: Install Argo Rollouts kubectl Plugin
Note: The plugin just provides better visualization, convenience commands, and features like the dashboard - it's not required to view or manage rollouts.

```bash
# Using Homebrew
brew install argoproj/tap/kubectl-argo-rollouts

# Verify installation
kubectl argo rollouts version
```

Example output from `kubectl argo rollouts version`
```bash
kubectl-argo-rollouts: v1.8.3+49fa151
  BuildDate: 2025-06-04T22:19:21Z
  GitCommit: 49fa1516cf71672b69e265267da4e1d16e1fe114
  GitTreeState: clean
  GoVersion: go1.23.9
  Compiler: gc
  Platform: darwin/amd64
```

Step 3: Create a Sample API Application
Create a file called blue-green-rollout.yaml


Step 4: Deploy the Application (Blue)
```bash
# Apply the configuration
kubectl apply -f blue-green-rollout.yaml

# Watch the rollout status
kubectl argo rollouts get rollout api-rollout --watch

# Check services
kubectl get svc
```
example output of "kubectl argo rollouts get rollout api-rollout --watch"
```bash
Name:            api-rollout
Namespace:       default
Status:          ✔ Healthy
Strategy:        BlueGreen
Images:          hashicorp/http-echo:latest (stable, active)
Replicas:
  Desired:       3
  Current:       3
  Updated:       3
  Ready:         3
  Available:     3

NAME                                     KIND        STATUS     AGE  INFO
⟳ api-rollout                            Rollout     ✔ Healthy  45s  
└──# revision:1                                                      
   └──⧉ api-rollout-7c7cfd787b           ReplicaSet  ✔ Healthy  45s  stable,active
      ├──□ api-rollout-7c7cfd787b-9pmqs  Pod         ✔ Running  44s  ready:1/1
      ├──□ api-rollout-7c7cfd787b-rq856  Pod         ✔ Running  44s  ready:1/1
      └──□ api-rollout-7c7cfd787b-zqc59  Pod         ✔ Running  44s  ready:1/1
Name:            api-rollout
Namespace:       default
Status:          ✔ Healthy
Strategy:        BlueGreen
Images:          hashicorp/http-echo:latest (stable, active)
Replicas:
  Desired:       3
  Current:       3
  Updated:       3
  Ready:         3
  Available:     3
```

Step 5: Access Your API
```bash
# Get the active service URL
kubectl get svc api-service-active

# Test the API (use the EXTERNAL-IP or localhost if using port-forward)
kubectl port-forward svc/api-service-active 8080:80

# In another terminal
curl http://localhost:8080
# example response: "Blue Version v1.0"
```

Step 6: Deploy a New Version (Green)
Create blue-green-rollout-v2.yaml or update the existing file (blue-green-rollout.yaml)
```bash
# Apply the update
kubectl apply -f blue-green-rollout-v2.yaml

# Watch the rollout
kubectl argo rollouts get rollout api-rollout --watch
```
example output of ""
```bash
Name:            api-rollout
Namespace:       default
Status:          ॥ Paused
Message:         BlueGreenPause
Strategy:        BlueGreen
Images:          hashicorp/http-echo:latest (active, preview, stable)
Replicas:
  Desired:       3
  Current:       6
  Updated:       3
  Ready:         3
  Available:     3

NAME                                     KIND        STATUS     AGE  INFO
⟳ api-rollout                            Rollout     ॥ Paused   36m  
├──# revision:2                                                      
│  └──⧉ api-rollout-594c48b8b            ReplicaSet  ✔ Healthy  29s  preview
│     ├──□ api-rollout-594c48b8b-6zxpr   Pod         ✔ Running  29s  ready:1/1
│     ├──□ api-rollout-594c48b8b-b8vrz   Pod         ✔ Running  29s  ready:1/1
│     └──□ api-rollout-594c48b8b-gpcqg   Pod         ✔ Running  29s  ready:1/1
└──# revision:1                                                      
   └──⧉ api-rollout-7c7cfd787b           ReplicaSet  ✔ Healthy  36m  stable,active
      ├──□ api-rollout-7c7cfd787b-9pmqs  Pod         ✔ Running  36m  ready:1/1
      ├──□ api-rollout-7c7cfd787b-rq856  Pod         ✔ Running  36m  ready:1/1
      └──□ api-rollout-7c7cfd787b-zqc59  Pod         ✔ Running  36m  ready:1/1
```

Step 7: Preview and Promote

```bash
# Check preview service (new green version)
kubectl port-forward svc/api-service-preview 8081:80

# In another terminal, test preview
curl http://localhost:8081 # response should be "Green Version v2.0"

# Check active service (still blue version)
curl http://localhost:8080  # Should show "Blue Version v1.0"

# If satisfied, promote the green version to active
kubectl argo rollouts promote api-rollout

# Or if there's an issue, abort the rollout
kubectl argo rollouts abort api-rollout
# Note: aborting is degrading the Green rollout and
#       make its pods are terminated, so we cannot access them via 8081.
#       we have to re-apply the new Green rollout (e.g., `kubectl apply -f blue-green-rollout-v2.yaml`) and
#       redo port-forward.
```

example output after aborting "kubectl argo rollouts abort api-rollout"
```bash
Name:            api-rollout
Namespace:       default
Status:          ✖ Degraded
Message:         RolloutAborted: Rollout aborted update to revision 2
Strategy:        BlueGreen
Images:          hashicorp/http-echo:latest (stable, active)
Replicas:
  Desired:       3
  Current:       3
  Updated:       0
  Ready:         3
  Available:     3

NAME                                     KIND        STATUS        AGE  INFO
⟳ api-rollout                            Rollout     ✖ Degraded    49m  
├──# revision:2                                                         
│  └──⧉ api-rollout-594c48b8b            ReplicaSet  • ScaledDown  13m  preview,delay:passed
└──# revision:1                                                         
   └──⧉ api-rollout-7c7cfd787b           ReplicaSet  ✔ Healthy     49m  stable,active
      ├──□ api-rollout-7c7cfd787b-9pmqs  Pod         ✔ Running     49m  ready:1/1
      ├──□ api-rollout-7c7cfd787b-rq856  Pod         ✔ Running     49m  ready:1/1
      └──□ api-rollout-7c7cfd787b-zqc59  Pod         ✔ Running     49m  ready:1/1
```

example output after re-apply the new rollout (blue-green-rollout-v3.yaml)
```bash
curl http://localhost:8080 # response="Blue Version v1.0"
curl http://localhost:8081 # response="Green Version v3"
```

```bash
Name:            api-rollout
Namespace:       default
Status:          ॥ Paused
Message:         BlueGreenPause
Strategy:        BlueGreen
Images:          hashicorp/http-echo:latest (active, preview, stable)
Replicas:
  Desired:       3
  Current:       6
  Updated:       3
  Ready:         3
  Available:     3

NAME                                     KIND        STATUS        AGE  INFO
⟳ api-rollout                            Rollout     ॥ Paused      59m  
├──# revision:3                                                         
│  └──⧉ api-rollout-955875d8f            ReplicaSet  ✔ Healthy     20s  preview
│     ├──□ api-rollout-955875d8f-8k6rc   Pod         ✔ Running     20s  ready:1/1
│     ├──□ api-rollout-955875d8f-mvhrn   Pod         ✔ Running     20s  ready:1/1
│     └──□ api-rollout-955875d8f-rd85j   Pod         ✔ Running     20s  ready:1/1
├──# revision:2                                                         
│  └──⧉ api-rollout-594c48b8b            ReplicaSet  • ScaledDown  23m  delay:passed
└──# revision:1                                                         
   └──⧉ api-rollout-7c7cfd787b           ReplicaSet  ✔ Healthy     59m  stable,active
      ├──□ api-rollout-7c7cfd787b-9pmqs  Pod         ✔ Running     59m  ready:1/1
      ├──□ api-rollout-7c7cfd787b-rq856  Pod         ✔ Running     59m  ready:1/1
      └──□ api-rollout-7c7cfd787b-zqc59  Pod         ✔ Running     59m  ready:1/1
```

example output after promoting "kubectl argo rollouts promote api-rollout"
```bash
Name:            api-rollout
Namespace:       default
Status:          ✔ Healthy
Strategy:        BlueGreen
Images:          hashicorp/http-echo:latest (stable, active)
Replicas:
  Desired:       3
  Current:       3
  Updated:       3
  Ready:         3
  Available:     3

NAME                                    KIND        STATUS        AGE    INFO
⟳ api-rollout                           Rollout     ✔ Healthy     64m    
├──# revision:3                                                          
│  └──⧉ api-rollout-955875d8f           ReplicaSet  ✔ Healthy     5m31s  stable,active
│     ├──□ api-rollout-955875d8f-8k6rc  Pod         ✔ Running     5m31s  ready:1/1
│     ├──□ api-rollout-955875d8f-mvhrn  Pod         ✔ Running     5m31s  ready:1/1
│     └──□ api-rollout-955875d8f-rd85j  Pod         ✔ Running     5m31s  ready:1/1
├──# revision:2                                                          
│  └──⧉ api-rollout-594c48b8b           ReplicaSet  • ScaledDown  28m    delay:passed
└──# revision:1                                                          
   └──⧉ api-rollout-7c7cfd787b          ReplicaSet  • ScaledDown  64m
```
