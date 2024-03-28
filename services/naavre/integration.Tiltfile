load('ext://helm_remote', 'helm_remote')

helm_remote(
  'jupyterhub',
  repo_url='https://jupyterhub.github.io/helm-chart/',
  values=[
    './helm/values-integration.yaml',
    ],
  )

k8s_resource(
  'hub',
  labels=['NaaVRE-integration'],
  trigger_mode=TRIGGER_MODE_MANUAL,
  links=['https://naavre-dev.minikube.test/n-a-a-vre-integration/'],
  resource_deps=['vrepaas-vreapi', 'n-a-a-vre-dev'],
  )
k8s_resource(
  'user-scheduler',
  labels=['NaaVRE-integration'],
  resource_deps=['vrepaas-vreapi', 'n-a-a-vre-dev'],
  )
k8s_resource(
  'proxy',
  labels=['NaaVRE-integration'],
  trigger_mode=TRIGGER_MODE_MANUAL,
  resource_deps=['vrepaas-vreapi', 'n-a-a-vre-dev'],
  )
k8s_resource(
  'user-placeholder',
  labels=['NaaVRE-integration'],
  pod_readiness='ignore',
  resource_deps=['vrepaas-vreapi', 'n-a-a-vre-dev'],
  )
