# ####################################################
#    / __ \/  _/   |  /  |/  /   |  / | / /_  __/  _/
#   / / / // // /| | / /|_/ / /| | /  |/ / / /  / /  
#  / /_/ // // ___ |/ /  / / ___ |/ /|  / / / _/ /   
# /_____/___/_/  |_/_/  /_/_/  |_/_/ |_/ /_/ /___/ 
# credits :agupta@diamani.com | brock@diamanti.com
# ####################################################

# ####################################################
# Assumptions: MSSQL Management tools are already loaded
# NEEDED FILES:
# StorageClass: m3-stroageclass.yaml
# Deployment: mssql2019-deploy-previleged.yaml
# ####################################################

# Login to the Diamanti Cluster
# This will also authenticate to kubernetes as well.

dctl -s <Cluster vIP> login
dctl cluster status

# Check for an existing deployment
kubectl get ns

#  Create K8s Name Space
kubectl create ns mssql

# Create Storage Class with 3way Mirror
kubectl apply -f m3-storageclass.yaml

# Create a secret for a password before creating the mssql pod
kubectl create secret generic mssql --from-literal=SA_PASSWORD="Diamanti1!!!" -n mssql

# Deploy the container
kubectl create -f mssql2019-deploy-privileged.yaml -n mssql

# ####################################################
# Use the dynamic IP of the pod to access the database. 
# IP of the pod might change in case the pod is 
# recreated for any reason. This solution is OK for
# the experimental dev database but not for production.
# ####################################################

# Validate Deployment and display the IP address
kubectl get pods -o wide -n mssql

# Trubleshooting tools
kubectl describe pod mssql2019-<STRING> -n mssql

# ####################################################
# Clean Up
# ####################################################

# ### Quick Way
# Kill NS
kubectl delete ns mssql
# Delete Storage Class
kubectl delete sc high-m3

# ### Long way
# Delete deployment
kubectl delete -f mssql2019-deploy-privileged.yaml -n mssql
# Delete Secret
kubectl delete secrets mssql -n mssql
# Delete NameSpace
kubectl delete ns mssql
# Delete StorageClass
kubectl delete sc high-m3