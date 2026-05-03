Copy $HOME/.ssh to the working directory for convenient
```
cp -r $HOME/.ssh .
```
Download LSF CE from https://www.ibm.com/support/pages/where-do-i-download-lsf-community-edition and build the container
```
docker build -f Dockerfile.ubuntu24 -t ubuntu24-lsf-qrmi:dev .
```
Start the container. After docker run, you can use ctrl-P ctrl-Q to exit without stopping container. To get another shell, use docker exec.
```
docker run -it --cap-add=NET_RAW --name ubuntu24-lsf-qrmi ubuntu24-lsf-qrmi:dev
docker exec -it ubuntu24-lsf-qrmi /bin/bash
```
Now start installing LSF CE in the container
```
tar zxfv lsfsce10.2.0.15-armv8.tar.Z
cd lsfsce10.2.0.15-armv8/lsf
tar zxfv lsf10.1_no_jre_lsfinstall.tar.Z
cd lsf10.1_lsfinstall
```
Modify following in install.config
```
LSF_TOP="/home/lsfuser/lsfce"
LSF_ADMINS="lsfuser"
LSF_CLUSTER_NAME="lsfcluster"
LSF_MASTER_LIST="<hostname>"
LSF_TARDIR="/home/lsfuser/lsfsce10.2.0.15-armv8/lsf/"
```
Choose the single-user installation when asked, then set up environment and LSF
```
./lsfinstall -f ./install.config # Install single-user cluster
source /home/lsfuser/lsfce/conf/profile.lsf
lsadmin limstartup # Set up LSF
lsadmin resstartup # Set up LSF
badmin hstartup # Set up LSF
```
Test LSF installation
```
lsid
lsload
bhosts
```
Get the LSF-QPU repository and install hooks
```
git clone git@github.ibm.com:QCHPC/lsf-qpu.git
cp lsf-qpu/qrmi-esub-jobstarter.py $LSF_SERVERDIR
ln -s $LSF_SERVERDIR/qrmi-esub-jobstarter.py $LSF_SERVERDIR/esub.qrmi
ln -s $LSF_SERVERDIR/qrmi-esub-jobstarter.py $LSF_SERVERDIR/jobstarter.qrmi
echo "JOB_STARTER = $LSF_SERVERDIR/jobstarter.qrmi" >> $LSF_ENVDIR/lsbatch/lsfcluster/configdir/lsb.queues
```
Set up Python virtual environment and install required Python modules and QRMI. The installation of QRMI is missing the LSF-QPU instructions.
```
python3 -m venv pyenv # lsf-qpu scripts are expecting pyenv
source $HOME/pyenv/bin/activate
pip install --upgrade pip
pip install requests dotenv qrmi
cd lsf-qpu/examples
```
Create .env with the following environment variables
```
QRMI_IBM_QRS_IAM_APIKEY="<your API key>"
QRMI_IBM_QRS_SERVICE_CRN="<your service CRN>"
QRMI_IBM_QRS_ENDPOINT="https://quantum.cloud.ibm.com/api/v1"
QRMI_IBM_QRS_IAM_ENDPOINT="https://iam.cloud.ibm.com"
QRMI_IBM_QRS_SESSION_MODE="batch"
```
The following commands are listed in the LSF-QPU instructions but only the second works
```
bsub -Is -a "qrmi(".env", 128)" "env | grep QRMI" # Does not work???
bsub -Is -a "qrmi(".env", 128, ibm_marrakesh)" "env | grep QRMI" # Should return the following...
Job <...> is submitted to default queue <interactive>.
<<Waiting for dispatch ...>>
<<Starting on fcbeeac3cfd9>>
ibm_marrakesh_QRMI_IBM_QRS_IAM_ENDPOINT=https://iam.cloud.ibm.com
ibm_marrakesh_QRMI_IBM_QRS_ENDPOINT=https://quantum.cloud.ibm.com/api/v1
LSB_JOBNAME=env | grep QRMI
ibm_marrakesh_QRMI_IBM_QRS_SERVICE_CRN=<your service CRN>
ibm_marrakesh_QRMI_IBM_QRS_IAM_APIKEY=<your API key>
ibm_marrakesh_QRMI_IBM_QRS_SESSION_MODE=batch
QRMI_IBM_QRS_BEST_DEVICE=ibm_marrakesh
```
To run the sampler and estimator
```
bsub -Is -a "qrmi(".env", 128, ibm_marrakesh)" ./run_example.sh
```
Good time to commit changes, for example
```
docker commit ubuntu24-lsf-qrmi localhost/ubuntu24-lsf-qrmi:dev
```
