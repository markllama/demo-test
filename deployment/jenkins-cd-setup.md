# Object Creation Order

* rolebinding
* route
* deploymentconfig
* serviceaccount
* service
  * test
  * test-jnlp

213  oc-3.6 create -f 00_rolebindings-test_edit.yaml
  214  oc-3.6 create -f 01_services-test-jnlp.yaml 
  215  oc-3.6 create -f 03_routes-test.yaml 
  216  oc-3.6 create -f 04_serviceaccount-test.yaml
  217  oc-3.6 create -f 02_services-test.yaml 
  218  oc-3.6 create -f 05_dc-test.yaml
