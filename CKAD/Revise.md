* https://github.com/dgkanatsios/CKAD-exercises/blob/main/a.core_concepts.md#create-the-yaml-for-a-new-resourcequota-called-myrq-with-hard-limits-of-1-cpu-1g-memory-and-2-pods-without-creating-it
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#remove-the-app-label-from-the-pods-we-created-before
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#check-the-annotations-for-pod-nginx1
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#view-the-yaml-of-the-replica-set-that-was-created-by-this-deployment
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#autoscale-the-deployment-pods-between-5-and-10-targetting-cpu-utilization-at-80
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#create-a-job-named-pi-with-image-perl534-that-runs-the-command-with-arguments-perl--mbignumbpi--wle-print-bpi2000
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#create-a-job-but-ensure-that-it-will-be-automatically-terminated-by-kubernetes-if-it-takes-more-than-30-seconds-to-execute
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/c.pod_design.md#create-a-cron-job-with-image-busybox-that-runs-every-minute-and-writes-date-echo-hello-from-the-kubernetes-cluster-to-standard-output-the-cron-job-should-be-terminated-if-it-takes-more-than-17-seconds-to-start-execution-after-its-scheduled-time-ie-the-job-missed-its-scheduled-time
* Tips on https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#create-and-display-a-configmap-from-a-env-file
* configMapKeyRef vs configMapRef: https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#create-a-configmap-anotherone-with-values-var6val6-var7val7-load-this-configmap-as-env-variables-into-a-new-nginx-pod and https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#create-a-configmap-called-options-with-the-value-var5val5-create-a-new-nginx-pod-that-loads-the-value-from-variable-var5-in-an-env-variable-called-option
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#limit-ranges
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#create-resourcequota-in-namespace-one-with-hard-requests-cpu1-memory1gi-and-hard-limits-cpu2-memory2gi
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#delete-the-pod-you-just-created-and-mount-the-variable-username-from-secret-mysecret2-onto-a-new-nginx-pod-in-env-variable-called-username
* Check how variable name was created ("ssh-privatekey"): https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#delete-the-pod-you-just-created-and-mount-the-variable-username-from-secret-mysecret2-onto-a-new-nginx-pod-in-env-variable-called-username
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/d.configuration.md#generate-an-api-token-for-the-service-account-myuser
* While creating a pod use `--expose` to create a service along with pod: https://github.com/dgkanatsios/CKAD-exercises/blob/main/f.services.md#create-a-pod-with-image-nginx-called-nginx-and-expose-its-port-80
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/f.services.md#create-a-service-that-exposes-the-deployment-on-port-6262-verify-its-existence-check-the-endpoints
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/f.services.md#create-an-nginx-deployment-of-2-replicas-expose-it-via-a-clusterip-service-on-port-80-create-a-networkpolicy-so-that-only-pods-with-labels-access-granted-can-access-the-deployment-and-apply-it
* * https://github.com/dgkanatsios/CKAD-exercises/blob/main/g.state.md#create-a-second-pod-which-is-identical-with-the-one-you-just-created-you-can-easily-do-it-by-changing-the-name-property-on-podyaml-connect-to-it-and-verify-that-etcfoo-contains-the-passwd-file-delete-pods-to-cleanup-note-if-you-cant-see-the-file-from-the-second-pod-can-you-figure-out-why-what-would-you-do-to-fix-that
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/g.state.md#create-a-busybox-pod-with-sleep-3600-as-arguments-copy-etcpasswd-from-the-pod-to-your-local-folder
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/j.podman.md#build-and-see-how-many-layers-the-image-consists-of
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/j.podman.md#build-and-see-how-many-layers-the-image-consists-of
* https://github.com/dgkanatsios/CKAD-exercises/blob/main/j.podman.md#build-and-see-how-many-layers-the-image-consists-of
* 