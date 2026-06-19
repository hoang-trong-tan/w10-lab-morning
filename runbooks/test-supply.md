# check admission policy 
kubectl logs -n cosign-system deployment/policy-controller-webhook | grep "admissionreview/allowed"


# create pod test auto add limitrange
kubectl run test-limitrange --image=ghcr.io/hoang-trong-tan/w10-api:0.0.5 --labels="owner=team-b" -n payments 


