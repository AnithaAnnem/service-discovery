# POC: Dynamic Service Discovery with Consul & Envoy

## 1. Overview
This Proof of Concept (POC) demonstrates the transition from a **Static Infrastructure** (manual IP management) to a **Dynamic Infrastructure** (automated service discovery). 

By using **Consul**, we have created a "Live Phonebook" for our services. This allows **Envoy** to find applications using names instead of hardcoded IP addresses.

---

## 2. Roles & Responsibilities

| Component | Responsibility | Owner | Identity/Endpoint |
| :--- | :--- | :--- | :--- |
| **Edge Proxy** | Public Entry & SSL | **Abhishek** | `keycloak.opstree.dev` |
| **Service Discovery** | **Internal DNS & Health** | **Anitha** | `*.service.consul` |
| **Backend Apps** | Application Logic | **Divya** | Keycloak, Minio, Orchestrator |

---

## 3. How it Works (The Flow)

1. **Registration**: Anitha runs Ansible roles that register the apps (Keycloak/Minio/Orchestrator) into the Consul Catalog.
2. **Naming**: Consul automatically generates an internal name: `<service-name>.service.consul`.
3. **Discovery**: Abhishek configures Envoy to use these `.consul` names as "upstreams."
4. **Resolution**: When a user hits the public URL, Envoy asks Consul for the current IP of the service name and routes the traffic.



---

## 4. Internal DNS Mapping (Source of Truth)

These are the internal names that must be used inside the **Envoy Cluster Configuration**:

| Service | Real IP | Port | Consul Internal DNS Name |
| :--- | :--- | :--- | :--- |
| **Keycloak** | `192.168.8.30` | `8080` | `keycloak.service.consul` |
| **Orchestrator** | `192.168.8.34` | `8443` | `orchestrator.service.consul` |
| **Minio** | `192.168.8.18` | `8443` | `minio.service.consul` |

---

## 5. Key Benefits

* **No Static IPs**: If a server IP changes, Abhishek does **not** need to update Envoy. Consul updates the IP automatically.
* **Health Aware**: If Keycloak crashes, Consul marks it "Critical." Envoy will stop sending traffic to that node immediately.
* **Architecture Cleanup**: We are removing **Nginx** from the app servers. Envoy now talks directly to the application ports via Consul names.

---

## 6. Verification Commands

To confirm the setup is working, run these commands on the **Envoy Server**:

### Check DNS Resolution
```bash
# This should return the IP of the Keycloak server (192.168.8.30)
dig @127.0.0.1 -p 8600 keycloak.service.consul +short

7. Next Steps for Implementation
[ ] Anitha: Finish registering Orchestrator and Minio with HTTPS (8443) health checks.

[ ] Abhishek: Update envoy.yaml to use type: STRICT_DNS for all clusters.

[ ] Team: Perform a failover test (stop a service and verify Envoy stops routing to it).
