load('ext://helm_remote', 'helm_remote')


# Keycloak

helm_remote(
  'keycloak',
  repo_name='bitnami',
  repo_url='https://charts.bitnami.com/bitnami',
  version='15.1.6',
  values=[
    'helm_config/keycloak/values.yaml',
    ]
  )

k8s_resource('keycloak', labels=['Keycloak'],
             links=['https://naavre-dev.minikube.test/auth/'])
k8s_resource('keycloak-postgresql', labels=['Keycloak'])
k8s_resource('keycloak-keycloak-config-cli', labels=['Keycloak'])
