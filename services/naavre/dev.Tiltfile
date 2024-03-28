k8s_yaml(helm(
  './helm/n-a-a-vre-dev/',
  name='n-a-a-vre-dev',
  values=[
    './helm/values-dev.yaml',
    ],
  ))

k8s_resource(
  'n-a-a-vre-dev',
  labels=['NaaVRE-dev'],
  links=['https://naavre-dev.minikube.test/n-a-a-vre-dev/'],
  resource_deps=['vrepaas-vreapi'],
  )
