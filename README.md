# Introduction

## Corteza

Notice the particular version of Corteza **2023.9**: you can use more recent versions, the CRM module, however, is not available in the most recent versions. The current solution is installing **2023.9** and upgrading to the latest version.

## Docker compose

The following changes were made in the [original docker compose file](https://docs.cortezaproject.org/corteza-docs/2023.3/devops-guide/examples/deploy-online/multi-discovery-pgsql.html):

You may get the following warnings after starting *Corteza*:

```text
es-1         | [2025-04-29T14:17:05,744][WARN ][o.o.b.JNANatives         ] [es] Unable to lock JVM Memory: error=12 reason=Cannot allocate memory
es-1         | [2025-04-29T14:17:05,746][WARN ][o.o.b.JNANatives         ] [es] This can result in part of the JVM being swapped out.
es-1         | [2025-04-29T14:17:05,746][WARN ][o.o.b.JNANatives         ] [es] Increase RLIMIT_MEMLOCK, soft limit: 8388608, hard limit: 8388608
es-1         | [2025-04-29T14:17:05,746][WARN ][o.o.b.JNANatives         ] [es] These can be adjusted by modifying /etc/security/limits.conf, for 
...
```

according to the logs, you should set the following lines in */etc/security/limits.conf*

```yaml
opensearch soft memlock unlimited
opensearch hard memlock unlimited
```

Another option is to disable the requirement to lock the JVM memory:

```yaml
  es:
    image: opensearchproject/opensearch:1.3.0
    restart: on-failure
    networks: [ internal ]
    environment:
      - cluster.name=es-docker-cluster
      - node.name=es
      - cluster.initial_master_nodes=es
      - bootstrap.memory_lock=true # along with the memlock settings below, disables swapping
     # - OPENSEARCH_JAVA_OPTS=-Xms8000m -Xmx8000m # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
      - DISABLE_INSTALL_DEMO_CONFIG=true
      - DISABLE_SECURITY_PLUGIN=true
      - VIRTUAL_HOST=es.${DOMAIN}
      - LETSENCRYPT_HOST=es.${DOMAIN}
    # ulimits:
    #   memlock:
    #     # https://github.com/containerd/nerdctl/discussions/1941
    #     soft: -1
    #     hard: -1
    volumes:
      - ./data/es:/usr/share/elasticsearch/data
```

The database services is fully disabled, an *OVH* managed *PostgreSQL* instance is used.