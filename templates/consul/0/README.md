# Consul Cluster


### Info:

 This template creates a global Consul stack that uses RPC encryption with TLS and gossip encryption to secure connection between consul cluster nodes, configuration is generated with confd from Rancher metadata. 
 TLS is used to verify the authenticity of the servers and the clients using the verify_incoming and verify_outgoing options.

 The variables used in this template include:

- Certificates and keys for Consul nodes.
- CA certificate.
- 16-bytes, Base64 encoded gossip encryption key.
- ACL settings.
 

The templates uses three Docker images one as the main image and the others are sidekicks:

- [consul](https://github.com/romracer/consul-rancher).
- [consul-config](https://github.com/romracer/consul-config).
- [consul-registrator](https://github.com/gliderlabs/registrator).
 
### Usage:
 
 Select Consul from catalog.

 Enter the certificates and keys for consul nodes, ca certificates, encryption key, and ACL token (use uuidgen if necessary).

 Click deploy.
 
 The consul nodes will be bound to the Rancher managed host IPs.
