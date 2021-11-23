
I first setup and run slightly modified cilium benchmark locally
#https://docs.cilium.io/en/latest/operations/performance/benchmark/#test-configurations
steps:
1. setup test environment
    create hosts.ini
2. install required software
    2.1 install netperf
	ansible-playbook -i conf/hosts.ini playbooks/install-misc.yaml
    2.2 install kubeadm
	ansible-playbook -i conf/hosts.ini playbooks/install-kubeadm.yaml
    2.3 install kubenetbench
	ansible-playbook -i conf/hosts.ini playbooks/install-kubenetbench.yaml
3. Run Benchmarks for cilium
    3.1 Tunneling
	ansible-playbook -e mode=tunneling -i conf/hosts.ini playbooks/install-k8s-cilium.yaml
	ansible-playbook -e conf=vxlan -i conf/hosts.ini playbooks/run-kubenetbench.yaml

    3.2 Native Routing with kube-proxy
        ansible-playbook -e kubeproxyfree=disabled -e mode=directrouting -i conf/hosts.ini playbooks/install-k8s-cilium.yaml
        ansible-playbook -e conf=routing -i conf/hosts.ini playbooks/run-kubenetbench.yaml

    3.3 Native Routing
	ansible-playbook -e mode=directrouting -i conf/hosts.ini playbooks/install-k8s-cilium.yaml
	ansible-playbook -e conf=routing -i conf/hosts.ini playbooks/run-kubenetbench.yaml

    3.4 Native Routing with xdp
        ansible-playbook -e mode=directroutingwithxdp -i conf/hosts.ini playbooks/install-k8s-cilium.yaml
        ansible-playbook -e conf=routing -i conf/hosts.ini playbooks/run-kubenetbench.yaml

    3.5 Encryption
	ansible-playbook -e kubeproxyfree=disabled -e mode=directrouting -e encryption=yes -i conf/hosts.ini playbooks/install-k8s-cilium.yaml
	ansible-playbook -e conf=encryption-routing -i conf/hosts.ini playbooks/run-kubenetbench.yaml

    3.6 Baseline
	ansible-playbook -i conf/hosts.ini playbooks/reset-kubeadm.yaml
	ansible-playbook -i conf/hosts.ini playbooks/run-rawnetperf.yaml

4. Running benchmark for Mizar
   ansible-playbook -e mode=directrouting -i conf/hosts.ini playbooks/install-k8s-mizar.yaml
   # beforing running the planned regular benchmarks, I'll try with short one first
   ansible-playbook -e conf=routing -i conf/hosts.ini playbooks/run-kubenetbench-test.yaml
   # ansible-playbook -e conf=routing -i conf/hosts.ini playbooks/run-kubenetbench.yaml

5. Cleanup
Not needed in my case, because we don't use Packet resource here.

