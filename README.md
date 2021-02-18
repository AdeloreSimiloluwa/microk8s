# microk8s

1.  Set up your user name, password (Username and passwords set are strictly based on user preference)

`NEWUSER=demo`

`USERPASSWD=n15pgj6w`

2.  Some logging

BLUE=`tput setaf 4` 

RST=`tput sgr0`

function info {
	echo -e "[${BLUE}info${RST}] $@"}`
  
3. Allow SSH password authentication

`sudo sed -i 
"/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart`

4.	 Install Microk8s

`sudo snap install microk8s --classic --channel=1.19/stable`

5. 	Create a demo account

	`sudo useradd -m -s /bin/bash -p $(openssl passwd -crypt "$USERPASSWD") "$NEWUSER"`
  

6. 	Give NEWUSER perms to use microk8s

`sudo usermod -a -G microk8s "$NEWUSER"`

`sudo chown -f -R "$NEWUSER" /.kube`

7. 	Enable Microk8s feature

`sudo microk8s enable dns dashboard storage`

8. 	Update kube-apiserver flags

`echo -e "--service-account-signing-key-file=\${SNAP_DATA}/certs/serviceaccount.key\n--service-account-issuer=kubernetes.default.svc"  | sudo tee -a /var/snap/microk8s/current/args/kube-apiserver > /dev/null`

9.	Restart microk8s


`sudo microk8s stop`

`sudo microk8s start`

10. 	Create kubeconfig


`sudo microk8s.kubectl config view --raw | sudo tee /.kube/config >/dev/null`

`export KUBECONFIG=/.kube/config`

11. 	Enabe Kubeflow

	`Microk8s enable kubeflow`
  
  
`sudo su "$NEWUSER" -c "microk8s.kubectl port-forward svc/istio-ingressgateway 8081:80 -n istio-system& "`

Enable Ingress

`microk8s enable ingress`

Create an ingress file and apply it

`nano ingset.yaml`


apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
 namespace: istio-system
 name: http-ingress
spec:
 rules:
 - http:
     paths:
     - path: /
       backend:
         serviceName: istio-ingressgateway
         servicePort: 80

Note: Just incase you are not familiar with nano, after editing ctrl o to save , press enter and then ctrl x to exit


Apply ingress with 

`microk8s kubectl apply -f ingset.yaml`








