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
  'k8s-secret-creator': [
    'k8s-secret-creator',
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
    'traefik',
    'square-root-v2',
    'square-root-v3'
    ],
  }
default_groups = [
  'keycloak',
  'argo',
  'k8s-secret-creator',
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
include('services/k8s-secret-creator/Tiltfile')
include('services/vrepaas/Tiltfile')
include('services/naavre/image.Tiltfile')
include('services/naavre/dev.Tiltfile')
include('services/naavre/integration.Tiltfile')
include('services/minio/Tiltfile')
include('services/traefik/Tiltfile')
include('services/canary-example/Tiltfile')
include('services/kube-prometheus-stack/Tiltfile')
include('services/flagger/Tiltfile')
