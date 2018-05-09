# Running the whole framework in a single container

Within this section we will run locally the whole Framework inside the container. It contains the following application bundle:

>* [Perceval](https://github.com/chaoss/grimoirelab-perceval): retrieval of data from data sources
>* [Perceval (bundle for OPNFV)](https://github.com/chaoss/grimoirelab-perceval-opnfv)
>* [Perceval (bundle for Mozilla)](https://github.com/chaoss/grimoirelab-perceval-mozilla)
>* [Perceval (bundle for Puppet)](https://github.com/chaoss/grimoirelab-perceval-puppet)
>* [KingArthur](https://github.com/chaoss/grimoirelab-kingarthur): batch processing for massive retrieval
>* [Elk](https://github.com/chaoss/grimoirelab-elk): storage and enrichment of data
>* [GrimoireLab Toolkit](https://github.com/chaoss/grimoirelab-toolkit): common utilities
>* [SortingHat](https://github.com/chaoss/grimoirelab-sortinghat): identity management
>* [Mordred](https://github.com/chaoss/grimoirelab-mordred): orchestration
>* [Sigils](https://github.com/chaoss/grimoirelab-sigils): reporting
>* [Manuscripts](https://github.com/chaoss/grimoirelab-manuscripts): visualizations and dashboards
>* [Bestiary](https://github.com/chaoss/grimoirelab-bestiary): web-based user interface to manage repositories and projects for Mordred
>* [Hatstall](https://github.com/chaoss/grimoirelab-hatstall): web-based user interface to manage SortingHat identities
>* [Tutorial](https://github.com/chaoss/grimoirelab-tutorial)
>* [GrimoireLab as a whole](https://github.com/chaoss/grimoirelab)

We will file also in this folder the files needed for it to run in standalone mode:

```
.
├── Dockerfile
├── Dockerfile-full
├── Dockerfile-installed
├── entrypoint-full.sh
├── identities.yaml
├── menu-grimoirelab.yaml
├── menu.yaml
├── mordred-dashboard.cfg
├── mordred-infra.cfg
├── mordred-override.cfg
├── mordred-project.cfg
├── orgs.json
├── projects.json
├── README.md
├── sg_roles_mapping.yml
└── stage
```

Then, we need firs to generate a file with the your own github token:

```
$ cat mytoken.cfg
[github]
api-token=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

So with that, we can execute the container:

```
$ docker run -p 127.0.0.1:5601:5601 \
    -v $(pwd)/mytoken.cfg:/override.cfg \
    -t grimoirelab/full
```
