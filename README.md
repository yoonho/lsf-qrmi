# lsf-qrmi

The "real" LSF-QRMI project is located at https://github.ibm.com/QCHPC/lsf-qpu/. This repository documents how I was able to use the code from https://github.ibm.com/QCHPC/lsf-qpu/ to run jobs on IBM Quantum systems using LSF Community Edition (free) in a container.

While https://github.ibm.com/QCHPC/lsf-qpu/ does not follow the usual integration pattern, it does have some interesting resource discovery and scheduling features. An LSF LIM (Load Information Manager) plugin can provide metrics about quantum systems. The LSF user can then specify constraints against these metrics when starting a quantum job. For example, the LIM plugin collects the number of pending jobs. The user can specify that his/her quantum job should run on a quantum system with less than 100 pending jobs using the following: ```bsub -R "select[pending_jobs < 100]" quantum_job.py```. Without the LIM plugin, the user can specify the number of required qubits and the quantum system as follows: ```bsub -Is -a "qrmi(".env", 128, ibm_marrakesh)" quantum_job.sh```.

The usual integration pattern is acquire-execute-release. The current implementation skips the acquire and release. Instead, the pre-execution hook (```esub```) only sets up the QRMI environment variables. It does not acquire the quantum resource or check its accessibility. LSF does provide a post-execution hook (```epilog```).
