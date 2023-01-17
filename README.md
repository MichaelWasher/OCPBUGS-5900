# Reproduction for OCPBUGS-5900

This repository can be used to recreate the OpenShift OLM issue described in OCPBUGS-5900.

The OLM creates RBAC policies for CSVs/InstallPlans based on the OperatorGroup definitions but the RBAC policy names are generated based on the operator name. Operators installed into differing namespaces with overlapping OperatorGroups will result in the RBAC policies applied to a single Operators serviceaccounts and not both.

To reproduce the issue:
~~~
# Apply the manifests (up to step 5)
for i in {1..5} ; do echo "oc apply -f step${i}"; oc apply -f "step${i}" ; done && sleep 60

# Collect the RoleBindings
rm pre-rolebindings.yaml
for NS in ns-a ns-b ; do echo "oc get rolebindings -o yaml -n $NS" ; oc get rolebindings -o yaml -n $NS | oc neat >> pre-rolebindings.yaml ; done

# Apply the final installation of Operator
oc apply -f step6

# Compare the `operand-deployment-lifecycle-manager.v2.0.0-operand--558775dc8c` RoleBinding in `ns-a` and see that this existed before the additional Operator was installed. But has been changed upon the new install
rm post-rolebindings.yaml
for NS in ns-a ns-b ; do echo "oc get rolebindings -o yaml -n $NS" ; oc get rolebindings -o yaml -n $NS | oc neat >> post-rolebindings.yaml ; done

diff pre-rolebindings.yaml post-rolebindings.yaml 
~~~

To remove the manifests:
~~~
for i in {6..1} ; do echo "oc delete -f step${i}"; oc delete -f "step${i}" ; done
~~~
