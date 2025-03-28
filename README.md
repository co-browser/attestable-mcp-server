# ➡️ attestable-mcp-server
<div align="center">

<strong>remotely attestable MCP server</strong>
</div>

## Overview

This project contains an [MCP Server](https://spec.modelcontextprotocol.io/specification/2024-11-05/server/) that is [remotely attestable](https://confidentialcomputing.io/2024/10/02/what-is-remote-attestation-enhancing-data-governance-with-confidential-computing/) by MCP clients. To achieve this, a trusted execution environment is used, which generates a certificate representing the currently-running code of the attestable-mcp-server. The attestable-mcp-server sends this certificate in the TLS handshake to an MCP client before connecting that proves the code it's running is the [same code built on github actions](https://github.com/co-browser/attestable-mcp-server/actions/runs/14132689556), and can be independently validated by building and running the code locally on emulated hardware or secure hardware; these values will be the same. The protocol used for client <-> server remote attestation is [RA-TLS](https://cczoo.readthedocs.io/en/latest/Solutions/rats-tls/index.html), an extension to TLS that adds machine and code specific measurements that can be verified by an MCP client.

The most important concept behind this RA-TLS certificate is that it embeds an [SGX quote](https://www.intel.com/content/dam/develop/public/us/en/documents/intel-sgx-dcap-ecdsa-orientation.pdf) in the standardized X.509 extension field with the [TCG DICE "tagged evidence" OID](https://trustedcomputinggroup.org/wp-content/uploads/DICE-Certificate-Profiles-r01_pub.pdf), which in turn embeds the SGX report and the complete Intel SGX certificate chain. In addition to the SGX quote, the certificate also contains the evidence claims, with the most important one being the "pubkey-hash" claim that contains the hash of the ephemeral public key (in DER format) generated by the TEE of the memory image of the running MCP server.

<strong>Features</strong>
- MCP Clients can remotely attest the code running on any MCP Server
- MCP Servers can optionally remotely attest MCP Clients

#### Producing Signed Artifacts
The github action script in this repo runs on a self-hosted github runner inside of a trusted execution environment (TEE). The action script will build a docker container containing the attestable-mcp-server and generate a signed attestation of the code running inside the TEE. This docker image is then signed by github. You can independently generate the same values with or without secure hardware, and query our running server and get the same values. 
  
## Dependencies
 - Intel SGX Hardware
 - Gramine
 - python 3.13
 - Ubuntu 22.04
 - Intel SGX SDK & PSW
   
## Quickstart

```
uv sync
docker build -t attestable-mcp-server .
gramine-sgx-gen-private-key
git clone https://github.com/gramineproject/gsc docker/gsc
cd docker/gsc
uv run ./gsc build-gramine --rm --no-cache -c ../gramine_base.config.yaml gramine_base
uv run ./gsc build -c ../attestable-mcp-server.config.yaml --rm attestable-mcp-server ../attestable-mcp-server.manifest
uv run ./gsc sign-image -c ../attestable-mcp-server.config.yaml  attestable-mcp-server "$HOME"/.config/gramine/enclave-key.pem
uv run ./gsc info-image gsc-attestable-mcp-server
```

## Starting Server on Secure Hardware
```
docker run -itp --device=/dev/sgx_provision:/dev/sgx/provision  --device=/dev/sgx_enclave:/dev/sgx/enclave -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket -p 8000:8000 --rm gsc-attestable-mcp-server
```

## Starting Server on local development machine
```
docker run -p 8000:8000 --rm gsc-attestable-mcp-server
```

## TODO
 - add MCP client demonstrating ra-tls
 - add intel-signed measurements from our [github action](https://github.com/co-browser/attestable-mcp-server/blob/main/.github/workflows/ci.yml) to this readme for simple independent verification

## Future Plans

 - JSON Web Key (JWK) attestation claim validation


### cobrowser.xyz
