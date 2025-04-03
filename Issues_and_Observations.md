# Showcasing the EKS Deployment on GitHub



## 🛠️ Issues Faced & Troubleshooting

### **Minikube Conflict**
**Issue:** Minikube was already running, causing conflicts in EKS pod deployment.
**Solution:**
```sh
kubectl delete pod <pod-name>
kubectl config use-context <eks-cluster-context>
```

### **aws-iam-authenticator Issue**
**Issue:** Incorrect installation of `aws-iam-authenticator` caused an interactive switch error.
**Solution:** Reinstall and verify the installation:
```sh
curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-02/bin/linux/amd64/aws-iam-authenticator
chmod +x aws-iam-authenticator
mv aws-iam-authenticator /usr/local/bin/
which aws-iam-authenticator
```
