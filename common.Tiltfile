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
        # deployments and services are already handled by Tilt's k8s_resource
        # Tilt will complain if we provide deployments / services as objects
        if r['kind'].lower() not in ['deployment', 'service', 'job']
    ]

    # add the namespace to the same Tilt group
    if create_namespace:
        all_objects.append('{}:namespace'.format(namespace))

    # deploy resources
    k8s_yaml(namespaced_resource_stream)

    # Tilt will automatically create a k8s_resource for each deployment 
    # --> avoid this by renaming all existing deployments
    for r in resources:
        if r['kind'].lower() in ['deployment', 'job']:
            if r['metadata']['name'] != workload:
                k8s_resource(workload=r['metadata']['name'], new_name=workload)

    # if no deployment exists, we have to create a workload manually
    num_deployments = len([
        1
        for r in resources 
        if r['kind'].lower() in ['deployment', 'job']
    ])

    if num_deployments > 0:
        k8s_resource(workload=workload, objects=all_objects, **kwargs)
    else:
        k8s_resource(new_name=workload, objects=all_objects, **kwargs)