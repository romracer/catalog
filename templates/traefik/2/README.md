# Traefik clustered/HA active load balancer (Experimental)

### Info:

 This template deploys a clustered/HA traefik active load balancers on top of Rancher. The configuration is generated and stored in a key-value store.  Backend configuration is derived from Rancher metadata and labels.
 It would be deployed in hosts with label traefik_lb=true.

### Config:

- host_label = "traefik_lb=true" # Host label where to run traefik service.
- http_port = 8080  # Port exposed to get access to the published services.
- https_port = 8443  # Port exposed to get secured access to the published services.
- admin_port = 8000  # Port exposed to get admin access to the traefik service.
- https_enable = <false | true | only>
  - false: Enable http enpoints and disable https ones.
  - true: Enable http and https endpoints.
  - only: Enable https endpoints and redirect http to https.
- acme_enable = false               # Enable/Disable acme traefik support.
- acme_email = "test@traefik.io"    # acme user email
- acme_ondemand = true              # acme ondemand parameter.
- acme_onhostrule = true            # acme onHostRule parameter.
- ssl_key # Paste your ssl key. *Required if you enable https
- ssl_crt # Paste your ssl crt. *Required if you enable https
- kv_backend = etcd                 # key-value store backend
- kv_address # Address to reach the KV store in ip:port format
- default_domain # Default domain appended to Rancher services. Can be overwritten with the traefik.domain label
- expose_default = false            # Expose rancher services by default. Can be overwritten with the traefik.enable=true/false label

### Service configuration labels:

Traefik labels can be added to your services, in order to get overwrite defaults in the dynamic traefik config.

- traefik.enable = <true | false> 
  - true: the service will be published as *service_name.stack_name.traefik_domain*
  - false: the service will not be published
- traefik.frontend.priority = <priority>     	  	# Override for frontend priority.
- traefik.protocol = < http | https	>		# Override the default http protocol
- traefik.backend.loadbalancer.sticky = < true | false	>	# Enable/disable sticky sessions to the backend
- traefik.domain = < domain.name >			# Domain name to append to services.  Defaults to default_domain
- traefik.port = < port >           # Port to expose throught traefik  
- traefik.path = < path >                   # Path rule. Multiple values separated by ","

Details for configuring the traefik rules can be found at: https://docs.traefik.io/basics/#frontends
This list is *not* exhaustive.  You can find more [here][traefik-docs] and [here][traefik-code].

WARNING: Only services with healthy state are added to traefik, so health checks are mandatory.

### Usage:

 Select Traefik from catalog. 
 
 Set the params.

 Click deploy.

 Services will be accessed throught hosts ip's whith $host_label: 

 - http://${service_name}.${stack_name}.${traefik.domain}:${http_port}
 - https://${service_name}.${stack_name}.${traefik.domain}:${https_port}
 
 or 
 
 - http://${stack_name}.${traefik.domain}:${http_port}
 - https://${stack_name}.${traefik.domain}:${https_port}

Note: To access the services, you need to create A or CNAMES dns entries for every one. 

[traefik-docs]: https://docs.traefik.io/toml/#rancher-backend
[traefik-code]: https://github.com/containous/traefik/blob/v1.2/provider/rancher.go
