# locks
Backing storage for Concourse CI [pool resource](https://github.com/concourse/pool-resource) 

## Directory structure
```
.
├── latest                  # resource pool
    ├── claimed             # directory containing claimed locks
    │   └── .gitkeep
    └── unclaimed           # directory containing unclaimed locks
        ├── .gitkeep
        └── web-integration # lock file (empty)       
```
### Adding a resource
```sh
resource_name=my-new-resource
mkdir -p $resource_name/{claimed,unclaimed}
touch $resource_name/{claimed,unclaimed}/.gitkeep
```
### Adding a named lock
```sh
lock_name=my-named-lock
touch $resource_name/unclaimed/$lock_name
```

## Usage
Define resource in pipeline
```yaml
resources:
- name: ((concourse-resource-name))
  type: pool
  source:
    uri: git@github.com:nbycomp/locks.git
    branch: master
    pool: ((resource-pool))
    private_key: ((private_key))
```
Use `acquire`/`claim` to obtain a resource:
```yaml
jobs:
- name: my-job
  plan:
  - put: ((concourse-resource-name))
    params: {claim: ((lock-name))} # or acquire: true to obtain a random lock
```
Use `ensure` to release the resource even if the task failed/crashed:
```yaml
jobs:
- name: integration-tests
  plan:
  - task: integration
    file: path/to/integration.yml    
    ensure:
      put: latest
      params: {release: latest}
```

## Manually releasing locks
If it is neccesary to manually release a lock, this can be done by moving the file from `claimed` to `unclaimed`, e.g.:
```sh
git mv latest/claimed/web-integration latest/unclaimed
git commit -m "Manually release lock"
git push
```

