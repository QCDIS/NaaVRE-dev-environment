naavre_dev_sync_steps = [
  sync('./submodules/NaaVRE/jupyterlab_vre/', '/live/py/jupyterlab_vre/'),
  sync('./submodules/NaaVRE/packages/', '/live/ts/packages/'),
  ]
naavre_dev_run_steps = [
  run('cd /live/ts/ && npx lerna run build --scope "@jupyter_vre/' + package +'"',
      trigger=['./submodules/NaaVRE/packages/' + package + '/'])
  for package in ['chart-customs', 'components', 'core', 'experiment-manager', 'notebook-containerizer',
                 'notebook-search', 'vre-menu', 'vre-panel']
  ]

docker_build(
  'qcdis/n-a-a-vre-dev',
  context='./submodules/NaaVRE',
  dockerfile='./submodules/NaaVRE/docker/vanilla/dev.Dockerfile',
  extra_tag=['qcdis/n-a-a-vre-dev:latest'],
  only=[
    './environment.yml',
    './lerna.json',
    './package.json',
    './jupyterlab_vre/',
    './jupyter-config/',
    './setup.py',
    './README.md',
    './package.json',
    './packages/',
    './tsconfig-base.json',
    './docker/start-jupyter.sh',
    './docker/start-jupyter-venv.sh',
    './docker/start-jupyter-venv-dev.sh',
    './docker/init_script.sh',
    './docker/repo_utils',
    './docker/vanilla/.condarc',
    ],
  live_update=naavre_dev_sync_steps + naavre_dev_run_steps,
  )
