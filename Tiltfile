load('ext://namespace', 'namespace_create')

# limit the number of parallel updates
update_settings(max_parallel_updates=2)

# specify all Tiltfiles that should be used for local deployment
# use comments to select services to improve speed
apps = [
    # 'nerdd-module' : 'https://github.com/molinfo-vienna/nerdd-module.git',
    # 'nerdd-link':'https://github.com/molinfo-vienna/nerdd-link.git',
    # 'nerdd-jobs',
    # 'nerdd-backend'
    'nerdd-frontend',
    # 'cypstrate' : 'https://github.com/molinfo-vienna/cypstrate.git',
    # 'cyplebrity',
    # 'hitdexter',
    # 'np-scout',
    # 'skindoctor',
    # 'fame',
    # 'glory',
    # 'gloryx',

    # essential:
    'storage'
]

namespace_create('local')
k8s_resource(objects=['local'], labels=['infra'], new_name='namespace')

# load all Tiltfiles
for app in apps:
    path_app_tiltfile = './apps/{}/Tiltfile'.format(app)
    if os.path.exists(path_app_tiltfile):
        include(path_app_tiltfile)