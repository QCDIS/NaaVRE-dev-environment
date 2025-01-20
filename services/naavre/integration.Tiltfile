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
  
  )
k8s_resource(
  'user-scheduler',
  labels=['NaaVRE-integration'],
  
  )
k8s_resource(
  'proxy',
  labels=['NaaVRE-integration'],
  trigger_mode=TRIGGER_MODE_MANUAL,
  
  )
k8s_resource(
  'user-placeholder',
  labels=['NaaVRE-integration'],
  pod_readiness='ignore',
  
  )
