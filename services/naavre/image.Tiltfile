load('../../helpers.Tiltfile', 'get_custom_build_command')

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

custom_build(
  'qcdis/n-a-a-vre-dev',
  get_custom_build_command(path='./submodules/NaaVRE', file='./submodules/NaaVRE/docker/vanilla/dev.Dockerfile', extra_args=['-t', 'qcdis/n-a-a-vre-dev:latest']),
  [
    './submodules/NaaVRE/environment.yml',
    './submodules/NaaVRE/lerna.json',
    './submodules/NaaVRE/package.json',
    './submodules/NaaVRE/jupyterlab_vre/',
    './submodules/NaaVRE/jupyter-config/',
    './submodules/NaaVRE/setup.py',
    './submodules/NaaVRE/README.md',
    './submodules/NaaVRE/package.json',
    './submodules/NaaVRE/packages/',
    './submodules/NaaVRE/tsconfig-base.json',
    './submodules/NaaVRE/docker/start-jupyter.sh',
    './submodules/NaaVRE/docker/start-jupyter-venv.sh',
    './submodules/NaaVRE/docker/start-jupyter-venv-dev.sh',
    './submodules/NaaVRE/docker/init_script.sh',
    './submodules/NaaVRE/docker/repo_utils',
    './submodules/NaaVRE/docker/vanilla/.condarc',
    ],
  skips_local_docker=True,
  disable_push=True,
  live_update=naavre_dev_sync_steps + naavre_dev_run_steps,
  )
