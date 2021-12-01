---
layout: post
title:  "CKS"
categories: Certification
---


Allowed sources according to FAQ:
* review Documents installed by the distribution (man and /usr/share)
* https://kubernetes.io/docs/home/
* https://kubernetes.io/blog/
* https://github.com/kubernetes/
* https://aquasecurity.github.io/trivy/
* https://docs.sysdig.com/
* https://falco.org/docs/
* https://gitlab.com/apparmor/apparmor/-/wikis/Documentation

---

CKS Curriculum
1. Cluster Setup 10%
    * Use Network security policies to restrict cluster level access
      https://kubernetes.io/docs/concepts/services-networking/network-policies/
      https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/
      https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/#restricting-network-access

    * Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi)
```
     kubelet:
        Make sure --anonymous-auth is not set to true.
        Use Webhook mode for authn/authz.
     kube-apiserver:
        Make sure the --profiling=false flag is set.
        Ensure authorization mode does not include AlwaysAllow.
        Make sure authorization mode includes Node and RBAC.
     etcd:
        Make sure client-cert-auth is set to true.
```
    * Properly set up Ingress objects with security control
      https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/tls.md
      https://kubernetes.io/docs/concepts/services-networking/ingress/

    * Protect node metadata and endpoints
      https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/

    * Minimize use of, and access to, GUI elements
      https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

    * Verify platform binaries before deploying
      -

2. Cluster Hardening 15%
   * Restrict access to Kubernetes API
      disbale loopback for api server ? https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips
      `sudo sed -e '/insecure-port/s/^/#/g' -i /etc/kubernetes/manifests/kube-apiserver.yaml`
   * Use Role Based Access Controls to minimize exposure
      https://kubernetes.io/docs/reference/access-authn-authz/rbac/
   * Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones
      disble SA token for pods?
   * Update Kubernetes frequently
       kubeadm upgrade ?

3. System Hardening 15%
   * Minimize host OS footprint (reduce attack surface)
      Disable gui and remove not related packages
   * Minimize IAM roles
      Disable cloud roles?
   * Minimize external access to the network
      Egress Network Policies ?
   * Appropriately use kernel hardening tools such as AppArmor, seccomp
     https://kubernetes.io/docs/tutorials/clusters/apparmor/
     https://kubernetes.io/docs/tutorials/clusters/seccomp/

4. Minimize Microservice Vulnerabilities 20%
   * Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts
      https://kubernetes.io/docs/concepts/policy/pod-security-policy/
      https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
      https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/
   * Manage kubernetes secrets
      Encrypt secrets in etcd  ???
      https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
   * Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)
      k get runtimeclass
      https://kubernetes.io/docs/concepts/containers/runtime-class/
   * Implement pod to pod encryption by use of mTLS
      Linkerd, istio ?
5. Supply Chain Security 20%
   * Minimize base image footprint
      Update dockerfile ?
   * Secure your supply chain: whitelist allowed image registries, sign and validate images
     https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook
     https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
   * Use static analysis of user workloads (e.g. kubernetes resources, docker files)
      ??? falco ?? sysdig ? trivy
   * Scan images for known vulnerabilities
      trivy

6. Monitoring, Logging and Runtime Security 20%
   * Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities
       Falco ?
       https://falco.org/docs/getting-started/installation/
       https://falco.org/docs/getting-started/running/
   * Detect threats within physical infrastructure, apps, networks, data, users and workloads
   * Detect all phases of attack regardless where it occurs and how it spreads
   * Perform deep analytical investigation and identification of bad actors within environment
   * Ensure immutability of containers at runtime
      "readOnly" volume mount
      "ReadOnlyRootFilesystem" (securityContext, PSP) https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems
   * Use Audit Logs to monitor access
     https://kubernetes.io/docs/tasks/debug-application-cluster/audit/
