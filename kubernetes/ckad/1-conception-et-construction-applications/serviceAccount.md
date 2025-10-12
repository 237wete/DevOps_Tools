
* Creation
oc create "sa wso2svc-account"  

* Dotation de droits anyuid  
"oc adm policy add-scc-to-user anyuid -z wso2svc-account"
