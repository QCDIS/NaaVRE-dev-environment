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


# Argo

k8s_yaml('helm_config/argo/argo-sso-secret.yaml')
k8s_yaml('helm_config/argo/auth-vre-api.yaml')
helm_remote(
  'argo-workflows',
  repo_name='argo',
  repo_url='https://argoproj.github.io/argo-helm',
  version='0.33.1',
  values=[
    'helm_config/argo/values.yaml',
    ]
  )

k8s_resource('argo-workflows-server', labels=['Argo'],
             links=['https://naavre-dev.minikube.test/argowf/'])
k8s_resource('argo-workflows-workflow-controller', labels=['Argo'])


# VREPaaS

k8s_yaml(helm(
  './submodules/VREPaaS-helm-charts/',
  name='vrepaas',
  values=[
    'helm_config/vrepaas/values.yaml',
    ],
  ))

k8s_resource('vrepaas-vreapi', labels=['VREPaaS'],
             links=['https://naavre-dev.minikube.test/vre-api-test/api/',
                    'https://naavre-dev.minikube.test/vre-api-test/admin/'])
k8s_resource('vrepaas-vreapp', labels=['VREPaaS'],
             links=[ 'https://naavre-dev.minikube.test/vreapp/'])
k8s_resource('vrepaas-postgresql', labels=['VREPaaS'])

docker_build(
    'qcdis/vreapi',
    context='./submodules/VREPaaS',
    dockerfile='./submodules/VREPaaS/tilt/vreapis/Dockerfile',
    only=['./vreapis/'],
    entrypoint='/app/entrypoint.dev.sh',
    live_update=[
        sync('./submodules/VREPaaS/vreapis', '/app'),
        run('cd /app && /opt/venv/bin/python manage.py makemigrations'),
        run('cd /app && /opt/venv/bin/python manage.py migrate'),
        run('cd /app && /opt/venv/bin/pip install -r requirements.txt',
             trigger='./submodules/VREPaaS/vreapis/requirements.txt'),
    ],
)
docker_build(
    'qcdis/vreapp',
    context='./submodules/VREPaaS',
    dockerfile='./submodules/VREPaaS/tilt/vre-panel/Dockerfile',
    only=['./vre-panel/'],
    live_update=[
        sync('./submodules/VREPaaS/vre-panel', '/app'),
        run('cd /app && npm install',
            trigger=['./submodules/VREPaaS/vre-panel/package.json'])
    ],
)


# NaaVRE

helm_remote(
  'jupyterhub',
  repo_url='https://jupyterhub.github.io/helm-chart/',
  values=[
    './helm_config/n-a-a-vre/values.yaml',
    ],
  )

k8s_resource('hub', labels=['NaaVRE'],
             links=['https://naavre-dev.minikube.test/n-a-a-vre/'])
k8s_resource('user-scheduler', labels=['NaaVRE'])
k8s_resource('proxy', labels=['NaaVRE'])
k8s_resource('hook-image-puller', labels=['NaaVRE'])
k8s_resource('hook-image-awaiter', labels=['NaaVRE'])
k8s_resource('user-placeholder', labels=['NaaVRE'], pod_readiness='ignore')
k8s_resource('continuous-image-puller', labels=['NaaVRE'])
