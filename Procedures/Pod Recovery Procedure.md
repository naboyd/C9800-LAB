# Pod Recovery Procedure

If a student does something to a pod then clone the back up pod to their pod number and re-initialize the configuration.
Use the Pod Initialization file to do that,   There are a few steps.  

1. Open the WLC-Initialization.txt file, search and replace <POD#> with the number of the pod you are repairing.
2. Clone the X-C9800-PODX file to the C9800-POD0(Pod number you are repairing)
3. Power up the pod and answer no to config wizard , enter config mode and type in the first section of the pod initialization file you created in step 1.
4. ssh to the pod , and finish out the config file.

This should put the pod in a ready state.  make sure you save the initialized pod to C9800-POD(pod #)-Config for reset purposes.