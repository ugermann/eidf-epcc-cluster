# Troubleshooting FAQ

### Q: `The connection to the server 10.22.11.239 was refused - did you specify the right host or port?` when using `kubectl`

The connection refusal has been resolved for 10.22.11.239. If this reoccurs for any reason you can switch to an alternative endpoint:

`kubectl config get-contexts`

will give output like:

```
CURRENT NAME CLUSTER AUTHINFO NAMESPACE
* eidf-general-prod eidf-general-prod eidf-general-prod informatics
eidf-general-prod-gpu1-vm01 eidf-general-prod-gpu1-vm01 eidf-general-prod
eidf-general-prod-gpu2-vm01 eidf-general-prod-gpu2-vm01 eidf-general-prod
eidf-general-prod-gpu2-vm02 eidf-general-prod-gpu2-vm02 eidf-general-prod
eidf-general-prod-gpu4-vm01 eidf-general-prod-gpu4-vm01 eidf-general-prod
eidf-general-prod-gpu4-vm03 eidf-general-prod-gpu4-vm03 eidf-general-prod
eidf-general-prod-gpu4-vm05 eidf-general-prod-gpu4-vm05 eidf-general-prod
eidf-general-prod-gpu4-vm07 eidf-general-prod-gpu4-vm07 eidf-general-prod
eidf-general-prod-gpu4-vm13 eidf-general-prod-gpu4-vm13 eidf-general-prod
```

To access if any endpoint is down you can use another context:

`kubectl --context eidf-general-prod-gpu1-vm01 get pods -n informatics`