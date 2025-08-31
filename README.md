# ClickHouse Cluster Ansible Role

Ansible role for deploying and configuring ClickHouse cluster with high availability.

## Components

- **ClickHouse Server** - main database
- **ClickHouse Keeper** - cluster coordination (ZooKeeper replacement)
- **HAProxy** - load balancing
- **Keepalived** - IP-level failover

## Requirements

- Ansible 2.9+
- Ubuntu 18.04+ / CentOS 7+ / RHEL 7+
- Python 3.6+

## Variables

Main variables are defined in `defaults/main.yml`:

```yaml
# ClickHouse Configuration
clickhouse_version: "25.4"
clickhouse_user: "clickhouse"
clickhouse_group: "clickhouse"
clickhouse_data_dir: "/var/lib/clickhouse"
clickhouse_log_dir: "/var/log/clickhouse"
clickhouse_config_dir: "/etc/clickhouse-server"

# Cluster Configuration
clickhouse_cluster_name: "clickhouse_cluster"

# Network Configuration
clickhouse_http_port: 8123
clickhouse_tcp_port: 9000
clickhouse_interserver_port: 9009

# Keeper Configuration
keeper_enabled: true
keeper_port: 9181
keeper_raft_port: 9234

# HAProxy Configuration
haproxy_enabled: true
haproxy_stats_port: 8404
haproxy_frontend_http_port: 8124
haproxy_frontend_tcp_port: 9001

# Keepalived Configuration
keepalived_enabled: false
keepalived_virtual_ip: "192.168.1.100"
keepalived_interface: "eth0"
keepalived_priority: 100

# TLS Configuration
tls_enabled: false
tls_cert_path: "files/clickhouse.crt"
tls_key_path: "files/clickhouse.key"
tls_ca_path: "files/ca.crt"

# Resource Limits
clickhouse_max_memory_usage: "8GB"
clickhouse_max_concurrent_queries: 100

# User Management
clickhouse_users: []
# Example:
# clickhouse_users:
#   - name: "admin"
#     password: "admin_password"
#     profile: "default"
#     quota: "default"
#     networks:
#       - "::/0"
#   - name: "readonly"
#     password: "readonly_password"
#     profile: "readonly"
#     quota: "default"
#     networks:
#       - "192.168.1.0/24"
```

## Usage

### Basic usage

```yaml
- hosts: clickhouse_cluster
  roles:
    - clickhouse_cluster
```

### With custom variables

```yaml
- hosts: clickhouse_cluster
  vars:
    clickhouse_cluster_name: "my_cluster"
    clickhouse_version: "25.4"
    keeper_enabled: true
    haproxy_enabled: true
    keepalived_enabled: false
    tls_enabled: true
  roles:
    - clickhouse_cluster
```

### With tags

```yaml
- hosts: clickhouse_cluster
  roles:
    - role: clickhouse_cluster
      tags: clickhouse
```

## Tags

- `clickhouse` - ClickHouse installation and configuration
- `keeper` - ClickHouse Keeper installation and configuration
- `haproxy` - HAProxy installation and configuration
- `keepalived` - Keepalived installation and configuration
- `tls` - TLS configuration

## File Structure

```
├── defaults/main.yml          # Default variables
├── tasks/
│   ├── main.yml              # Main tasks
│   ├── clickhouse.yml        # ClickHouse tasks
│   ├── keeper.yml            # Keeper tasks
│   ├── haproxy.yml           # HAProxy tasks
│   ├── keepalived.yml        # Keepalived tasks
│   └── tls.yml               # TLS tasks
├── templates/
│   ├── clickhouse-config.xml.j2
│   ├── users.xml.j2
│   ├── keeper-config.xml.j2
│   ├── haproxy.cfg.j2
│   └── keepalived.conf.j2
├── vars/main.yml             # Role variables
├── files/                    # TLS certificates (optional)
│   ├── clickhouse.crt
│   ├── clickhouse.key
│   └── ca.crt
└── README.md
```

## Ports

- **8123** - ClickHouse HTTP interface
- **9000** - ClickHouse TCP interface
- **9009** - Interserver HTTP port
- **9181** - ClickHouse Keeper
- **8124** - HAProxy HTTP frontend (load balancer)
- **9001** - HAProxy TCP frontend (load balancer)
- **8404** - HAProxy statistics

## TLS Certificates

The role requires TLS certificates to be placed in the `files/` folder:
- `files/clickhouse.crt` - server certificate
- `files/clickhouse.key` - private key
- `files/ca.crt` - certificate authority certificate

If any of these files are missing, the role will fail with a clear error message.

To enable TLS, set `tls_enabled: true` in variables.

## Monitoring

- HAProxy statistics: `http://host:8404/stats`
- ClickHouse HTTP: `http://host:8123/ping`
- ClickHouse Keeper: `telnet host 9181`

## Service Management

The role uses the `service` module for ClickHouse and ClickHouse Keeper management, following the official ClickHouse documentation. Services are automatically enabled and started after configuration.

## Network Configuration

Both ClickHouse and ClickHouse Keeper are configured to listen on all interfaces (`::`) to enable proper cluster communication.
