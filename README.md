# ➡️ attestable-mcp-server
<div align="center">

<strong>Confidential MCP server</strong>
</div>

## Overview

This project contains a Confidential [MCP Server](https://spec.modelcontextprotocol.io/specification/2024-11-05/server/) running on [Gramine](https://github.com/gramineproject/gramine). 

Features: 
- Gramine runs the MCP server in a secure TEE: Intel SGX, solving the issue of Data Leak/Privacy Leak on MCP Servers.

- The MCP server implemented here is a Gdrive server which reads and searches files.

- There is the option of running the app in Local Development (No Gramine), In Gramine Direct Mode (No SGX security) and on SGX Hardware
  
## Dependencies
 - Intel SGX Hardware
 - Gramine
 - python 3.13
 - Ubuntu 22.04
 - Intel SGX SDK & PSW

## Local Development
```
uv sync
uv run python src/attestable_mcp_server/server.py --isDev  
```

## Production

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

Note: Build gramine_base only once.

## Starting Server in Direct Mode
```
docker run -it --rm \
  --security-opt seccomp=unconfined \
  --cap-add=SYS_PTRACE \
  --cap-add=SYS_ADMIN \
  -e GRAMINE_MODE=direct \
  gsc-attestable-mcp-server
```


## Starting Server on Secure Hardware
```
docker run -itp --device=/dev/sgx_provision:/dev/sgx/provision  --device=/dev/sgx_enclave:/dev/sgx/enclave -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket -p 8000:8000 --rm gsc-attestable-mcp-server
```
