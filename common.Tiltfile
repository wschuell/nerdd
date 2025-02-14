load('ext://namespace', 'namespace_create', 'namespace_inject')

def kustomize_resource(workload, kustomization_path, namespace, create_namespace=False, **kwargs):
    # create the namespace
    if create_namespace:
        namespace_create(namespace)

    # all resources should be loaded to the given namespace and grouped in Tilt
    # -> load all resources with kustomize
    # -> inject the namespace 'strimzi'
    # -> collect names of all resources using decode_yaml_stream
    resource_stream = kustomize(kustomization_path)
    namespaced_resource_stream = namespace_inject(resource_stream, namespace)
    resources = decode_yaml_stream(namespaced_resource_stream)
    all_objects = [
        '%s:%s:%s' % (r['metadata']['name'], r['kind'], r['metadata']['namespace']) 
        for r in resources
        # deployments, jobs and services are already handled by Tilt's k8s_resource
        # Tilt will complain if we provide deployments / services as objects
        if r['kind'].lower() not in ['deployment', 'job', 'service']
    ]

    # add the namespace to the same Tilt group
    if create_namespace:
        all_objects.append('{}:namespace'.format(namespace))

    # deploy resources
    k8s_yaml(namespaced_resource_stream)

    # Tilt will automatically create a k8s_resource for each deployment, job and service. 
    workloads = [
        r['metadata']['name']
        for r in resources 
        if r['kind'].lower() in ['deployment', 'job']
    ]

    for workload in workloads:
        k8s_resource(workload=workload, **kwargs)

    # put all other resources (ServiceAccounts, RoleBindings, etc.) in a single workload
    if len(all_objects) > 0:
        if workload in workloads:
            k8s_resource(workload=workload, objects=all_objects, **kwargs)
        else:
            k8s_resource(new_name=workload, objects=all_objects, **kwargs)