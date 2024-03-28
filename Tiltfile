config.define_string_list("groups-to-run", args=True)
cfg = config.parse()
groups = {
  'keycloak': [
    'keycloak',
    'keycloak-postgresql',
    'keycloak-keycloak-config-cli',
    ],
  'argo': [
    'argo-workflows-server',
    'argo-workflows-workflow-controller',
    ],
  'vrepaas': [
    'vrepaas-vreapi',
    'vrepaas-vreapp',
    'vrepaas-postgresql',
    ],
  'dev': [
    'n-a-a-vre-dev',
    ],
  'integration': [
    'n-a-a-vre-dev',
    'hub',
    'proxy',
    'user-placeholder',
    'user-scheduler',
    ],
  'extras': [
    'minio',
    ],
  }
default_groups = [
  'keycloak',
  'argo',
  'vrepaas',
  ]
groups_to_run = cfg.get('groups-to-run', [])
resources = []
for group in default_groups + groups_to_run:
  if group in groups:
    resources += groups[group]
  else:
    print('Unknown group: ' + group)
print(resources)
config.set_enabled_resources(resources)

include('services/keycloak/Tiltfile')
include('services/argo/Tiltfile')
include('services/vrepaas/Tiltfile')
include('services/naavre/image.Tiltfile')
include('services/naavre/dev.Tiltfile')
include('services/naavre/integration.Tiltfile')
include('services/minio/Tiltfile')
