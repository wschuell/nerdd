load('ext://namespace', 'namespace_create', 'namespace_inject')

def get_resource_mapping(yaml=None):
    if yaml == None: # note: starlark doesn't support "is None"
        # If yaml is None, we assume that the user wants to retrieve the currently available 
        # resources. We use kubectl api-resources to get the list of resources.
        blob = local('kubectl api-resources', quiet=True)

        # The command kubectl api-resources returns a list of all resources that Kubernetes knows 
        # about. The result looks like this:
        # NAME         SHORTNAMES  APIGROUP  NAMESPACED   KIND
        # bindings                           true         Binding
        # pods         po          v1        true         Pod
        # deployments  deploy      apps/v1   true         Deployment
        # ...
        output = str(blob)

        # split the output into lines
        lines = output.splitlines()

        # parse the column names from the first line
        # Note: we can't split column values by space, because some values are empty (spaces). For 
        # that reason, we need to find the positions of the columns first.
        positions = []
        in_column = True
        start = 0
        for i in range(len(lines[0])):
            char = lines[0][i]
            if char != ' ' and not in_column:
                positions.append((start, i))
                start = i
                in_column = True
            elif char == ' ' and in_column:
                in_column = False
        max_line_length = max([len(line) for line in lines])
        positions.append((start, max_line_length)) # last column

        column_names = [
            lines[0][start:end].strip().lower()
            for start, end in positions
        ]

        # parse the rest of the lines
        resources = [
            dict(
                zip(
                    column_names, 
                    [
                        line[start:end].strip()
                        for start, end in positions
                    ]
                )
            )
            for line in lines[1:]
        ]
    else:
        # read yaml 
        if type(yaml) == 'string':
            objects = read_yaml_stream(yaml)
        elif type(yaml) == 'blob':
            objects = decode_yaml_stream(yaml)
        else:
            fail('only takes string or blob, got: %s' % type(yaml))

        # filter CRDs
        crds = [
            obj
            for obj in objects
            if obj['kind'] == 'CustomResourceDefinition'
        ]

        # we only have to figure out NAME, SHORTNAMES, APIVERSION, NAMESPACED and KIND to be 
        # consistent with the output of kubectl api-resources
        resources = [
            dict(
                name=obj['spec']['names']['plural'],
                shortnames=obj['spec']['names']['shortNames'] if 'shortNames' in obj['spec']['names'] else '',
                apiversion=obj['spec']['group'] + '/' + version['name'],
                namespaced=obj['spec']['scope'] == 'Namespaced',
                kind=obj['spec']['names']['kind']
            )
            for obj in crds
            for version in obj['spec']['versions']
        ]

    # make it a dictionary
    resources_dict = {
        resource['kind']: resource
        for resource in resources
    }

    return resources_dict


def replace_namespace(yaml, new_namespace, forbidden_namespaces=['default', 'kube-system']):
    # read yaml 
    if type(yaml) == 'string':
        objects = read_yaml_stream(yaml)
    elif type(yaml) == 'blob':
        objects = decode_yaml_stream(yaml)
    else:
        fail('only takes string or blob, got: %s' % type(yaml))

    # get all resources (and merge)
    resources = dict(
        get_resource_mapping(),
        **get_resource_mapping(yaml)
    )

    # differentiate in namespaced and non-namespaced resources
    namespaced_resources = [
        resource['kind']
        for resource in resources.values()
        if resource['namespaced'] == 'true'
    ]

    def _r(obj, path):
        for p in path[:-1]:
            if p in obj:
                obj = obj[p]
            else:
                return

        # check if
        # a) the key does not exist (-> unspecified hints at patchable namespace)
        # b) the key does exist and is not in the forbidden namespaces (e.g. "default")
        if (path[-1] not in obj) or (obj[path[-1]] not in forbidden_namespaces):
            obj[path[-1]] = new_namespace


    # inject the namespace
    for obj in objects:
        kind = obj['kind']

        if kind in namespaced_resources:
            _r(obj, ['metadata', 'namespace'])

        #
        # SPECIAL CASES
        #

        if kind == 'Deployment': # templates in deployments
            _r(obj, ['spec', 'template', 'metadata', 'namespace'])
        elif kind == 'RoleBinding': # subjects in role bindings
            for subject in obj['subjects']:
                _r(subject, ['namespace'])

    return encode_yaml_stream(objects)

def fix_names(yaml):
    # read yaml 
    if type(yaml) == 'string':
        objects = read_yaml_stream(yaml)
    elif type(yaml) == 'blob':
        objects = decode_yaml_stream(yaml)
    else:
        fail('only takes string or blob, got: %s' % type(yaml))

    for o in objects:
        if 'metadata' in o and 'name' in o['metadata'] and ':' in o['metadata']['name']:
            print('Replacing ":" in name %s (namespace: %s, kind: %s)' % (
                o['metadata']['name'],
                o['metadata']['namespace'] if 'namespace' in o['metadata'] else 'default',
                o['kind']
            ))
            o['metadata']['name'] = o['metadata']['name'].replace(':', '-')

    return encode_yaml_stream(objects)

def convert_to_tilt_id(resource_object):
    namespace = (
        resource_object['metadata']['namespace'] 
        if 'namespace' in resource_object['metadata'] 
        else 'default'
    )

    return '%s:%s:%s' % (
        resource_object['metadata']['name'], 
        resource_object['kind'], 
        namespace
    )

def is_handled_by_tilt(resource_object, all_resources):
    # Tilt will automatically create a k8s_resource for specific resources:
    tilt_workload_resources = ['Deployment', 'Job', 'DaemonSet', "Service"]

    # special case: services are assigned by Tilt if there is a matching workload resource
    # if resource_object['kind'] == 'Service':
    #     workload_objects = [
    #         convert_to_tilt_id(r)
    #         for r in all_resources
    #         if r['kind'] in tilt_workload_resources
    #     ]

    #     for k in tilt_workload_resources:
    #         if convert_to_tilt_id(dict(resource_object, **{'kind': k})) in workload_objects:
    #             return True
    
    return resource_object['kind'] in tilt_workload_resources


def kustomize_resource(workload, kustomization_path, namespace, create_namespace=False, **kwargs):
    # all resources should be loaded to the given namespace and grouped in Tilt
    # -> load all resources with kustomize
    # -> inject the namespace to all resources
    # -> collect names of all resources using decode_yaml_stream
    resource_stream = kustomize(kustomization_path)
    resource_stream = replace_namespace(resource_stream, namespace)
    resource_stream = fix_names(resource_stream)
    resources = decode_yaml_stream(resource_stream)

    all_objects = [
        convert_to_tilt_id(r)
        for r in resources
        # Specific resources are already handled by Tilt's k8s_resource. Tilt will complain if 
        # we provide them in k8s_resource calls so we filter them here.
        if not is_handled_by_tilt(r, resources)
    ]

    # add the namespace to the same Tilt group
    namespace_object = '{}:Namespace:default'.format(namespace)
    if create_namespace and namespace_object not in all_objects:
        # create the namespace
        namespace_create(namespace)
        all_objects.append(namespace_object)

    # deploy resources
    k8s_yaml(resource_stream)

    # Tilt will automatically create a k8s_resource for all resources listed in tilt_workload_resources
    # --> identify the workloads already created by Tilt
    # --> add the specified dependencies, labels, etc. to it.

    # identify
    workloads = [
        r['metadata']['name']
        for r in resources 
        if is_handled_by_tilt(r, resources) and r['kind'] != "Service"
    ]

    # Tilt might have already created workloads automatically. If there is already a workload 
    # with the name specified in the parameter "workload", we add all objects to this workload.
    # Otherwise, we create a new workload.
    create_extra_workload = (workload not in workloads) and (len(all_objects) > 0)

    if create_extra_workload:
        k8s_resource(new_name=workload, objects=all_objects, **kwargs)

    for w in workloads:
        # The automatically created workloads might depend on other resources (e.g. namespaces, 
        # service accounts, etc.) from all_objects. Since we put those in a different workload,
        # we have to add a resource dependency here.
        modified_kwargs = {}
        if create_extra_workload:
            modified_kwargs['resource_deps'] = [workload]
        elif w == workload or (workload not in workloads and w == workloads[0]):
            modified_kwargs = kwargs
            modified_kwargs['objects'] = modified_kwargs.get('objects', []) + all_objects

        # all workloads should be put under the same label
        modified_kwargs['labels'] = kwargs.get('labels')

        k8s_resource(workload=w, **modified_kwargs)

    return resources