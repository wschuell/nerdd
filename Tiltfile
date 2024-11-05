

# update_settings(k8s_upsert_timeout_secs=60)
update_settings(max_parallel_updates=2)

modules = {
    # 'nerdd-module' : 'https://github.com/molinfo-vienna/nerdd-module.git',
    # 'nerdd-link':'https://github.com/molinfo-vienna/nerdd-link.git',
    # 'nerdd-jobs',
    # 'nerdd-backend'
    # 'nerdd-frontend',
    # 'cypstrate' : 'https://github.com/molinfo-vienna/cypstrate.git',
    # 'cyplebrity',
    # 'hitdexter',
    # 'np-scout',
    # 'skindoctor',
    # 'fame',
    # 'glory',
    # 'gloryx',
}

# # checkout all modules
# for directory, repository_url in modules.items():
#     checkout_dir = './{}'.format(directory)
#     if not os.path.exists(checkout_dir):
#         git_checkout(
#             repository_url, 
#             checkout_dir=checkout_dir
#         )

# load all Tiltfiles
for file in listdir('k8s', recursive=True):
    if file.endswith('Tiltfile'):
        include(file)