Autoscaling Strategy for Frontend and Backend Services

1. Horizontal Pod Autoscaling (HPA)

Possibility: Horizontal Pod Autoscaling provides the ability to scale pods automatically based on CPU or memory utilization metrics. For both the frontend and backend services, we are leveraging CPU usage as the key metric for scaling.

This allows us to increase or decrease the number of running pods based on real-time application demand, ensuring consistent performance under load without over-provisioning resources during periods of low traffic.

HPA Strategy
Scale Metrics: We use CPU utilization as the primary metric to trigger scaling. Setting the threshold at 70% ensures that the application scales before hitting resource limits, allowing it to gracefully handle spikes in traffic.

Replica Strategy:

Minimum Replicas: Setting the minimum to 2 ensures high availability at all times, even when traffic is low.
Maximum Replicas: By setting the maximum to 10, we ensure that the system can handle traffic surges effectively while controlling resource costs.
Target Utilization: The targetCPUUtilizationPercentage of 70% balances responsiveness and cost-efficiency. Pods will scale when the CPU usage exceeds 70%, preventing performance degradation during traffic spikes.


2. Cluster Autoscaling
Possibility: While HPA scales the number of pods, Cluster Autoscaling adjusts the number of nodes in the Kubernetes cluster to accommodate the changing demand. This ensures that the cluster itself has enough resources to handle the increased number of pods during traffic spikes.

Cluster Autoscaling Strategy
Scaling Limits: Node groups should have well-defined scaling limits. For example, a cluster can be configured to scale between 2 and 10 nodes, depending on the application’s needs.

Node Size and Resource Distribution: Ensure that each node is appropriately sized for the workload and that node pools are optimized for different types of workloads.

Resource Requests and Limits: Every deployment (frontend and backend) has defined resource requests and limits. These parameters help the Cluster Autoscaler decide when to add or remove nodes based on actual resource consumption.