# Lesson -  Introduction

As more organizations embrace containerization and adopt Kubernetes, they reap the benefits of platform scalability, application portability, and optimized infrastructure utilization. However, with this shift comes a new set of security challenges related to enabling connectivity for applications in heterogeneous environments.

In this workshop, we'll examine how the Calico Egress Gateway can help mitigate these issues by providing robust access control. By using Calico Egress Gateway, enterprises can secure communication from their Kubernetes workloads to the internet, 3rd party applications and networks while maintaining a high level of security.

The Calico Egress Gateway enforces security policies to regulate traffic flowing out of the Kubernetes cluster, providing granular control over egress traffic. This ensures that only authorized traffic is allowed to leave the cluster, mitigating the risks associated with unauthorized outbound traffic.

## Egress Security Challenges

For enterprises developing cloud-native applications with containers and Kubernetes, a frequent requirement is to connect to a database server hosted either on-prem or in the cloud, which is safeguarded by a network-based firewall. Since workloads with Kubernetes are dynamic without a fixed IP address, enabling such connectivity from workloads requires opening up a range of IP addresses resulting in security exposure that malicious actors can exploit to gain unauthorized access to the database via the Kubernetes clusters. Kubernetes clusters have the following limitations that make it challenging to enforce granular egress access control. 

Workload IPs (Pod CIDR) are typically not advertised between heterogeneous networks; doing so requires unique Pod CIDRs for each cluster that most organizations cannot afford due to limited IPv4 address space. As a result, traffic is NATed to the node's IP, which is non-deterministic.
Unlike virtual machines (VMs), pod IPs in Kubernetes are ephemeral and cannot be used to identify a workload.

Security teams have invested years protecting critical assets by implementing zone-based security using network firewalls. Existing security processes require connectivity between assets in disparate environments to be routed, restricted, and logged by firewalls. Regulatory requirements such as PCI-DSS mandates the archival of firewall logs for audit, security incident detection, and response. Organizations risk non-compliance when Kubernetes workloads communicate with external services and fail to meet these requirements. 
