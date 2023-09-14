This folder contains two yaml documents that will deploy nginx along with a network policy.

1. What cluster configuration prerequisites must be in place to use this network policy?

2. What configuration would need to be present on a pod so that it could connect to the nginx service with the policy applied?<br>
_NOTE:_<br>
You can run a pod to test the connection using the following command:<br>
`kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2`
<br>_Hint, you can supply the required configuration with a modification to the above test command_