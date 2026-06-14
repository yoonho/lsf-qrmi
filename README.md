# lsf-qrmi

The "real" LSF-QRMI project is located at https://github.ibm.com/QCHPC/lsf-qpu/. This repository documents how I was able to use the code from https://github.ibm.com/QCHPC/lsf-qpu/ to run jobs on IBM Quantum systems using LSF Community Edition (free) in a container (see INSTALL.md), move the integration to the usual acquire-execute-release pattern, and incorporate the QRMI resource discovery feature released in version 0.17.0.

The integration performs the acquire-execute in a jobstarter hook (jobstarter.qrmi) and the release in a postexec hook (postexec.qrmi). The hooks are expecting QRMI_QPU_RESOURCES and optionally QRMI_QPU_RESOURCES_FILTER and LSF_QRMI_DEBUG to be defined when running bsub (example below).

```
QRMI_QPU_RESOURCES=ibm_inst1,ibm_marrakesh QRMI_QPU_RESOURCES_FILTER="name=ibm_f*" LSF_QRMI_DEBUG=1 bsub -Is ./run_example.sh
```

```ibm_inst1``` is a dynamic resource that will be filtered by the expression ```name=ibm_f*``` that would filter out quantum system names such as ```ibm_kingston```. See https://github.com/qiskit-community/qrmi/pull/142 for additional details about dynamic resources. ```ibm_marrakesh``` is a static resource.

While https://github.ibm.com/QCHPC/lsf-qpu/ does not follow the usual integration pattern, it does have some interesting resource discovery and scheduling features. An LSF LIM (Load Information Manager) plugin can provide metrics about quantum systems. The LSF user can then specify constraints against these metrics when starting a quantum job. For example, the LIM plugin collects the number of pending jobs. The user can specify that his/her quantum job should run on a quantum system with less than 100 pending jobs using the following: ```bsub -R "select[pending_jobs < 100]" quantum_job.py```. Without the LIM plugin, the user can specify the number of required qubits and the quantum system as follows: ```bsub -Is -a "qrmi(".env", 128, ibm_marrakesh)" quantum_job.sh```.
