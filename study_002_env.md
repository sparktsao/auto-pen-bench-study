# Study 002: AutoPenBench Environment Creation and Variety Analysis

## Overview

This study analyzes how AutoPenBench creates and manages its testing environments, examining the infrastructure choices, variety mechanisms, and scalability approaches used to provide diverse penetration testing scenarios.

## Key Findings

### Environment Creation: **Docker-Based Containerized Infrastructure**

AutoPenBench uses a **sophisticated Docker-based architecture** with:

1. **Multi-layered Docker Compose orchestration**
2. **Automated container lifecycle management**
3. **Network isolation and IP address management**
4. **Programmatic environment generation**

## Infrastructure Architecture

### 1. Docker Compose Orchestration

#### Hierarchical Compose Structure
```
benchmark/machines/
├── docker-compose.yml              # Base Kali attacker machine
├── in-vitro/
│   ├── access_control/docker-compose.yml    # Access control VMs
│   ├── web_security/docker-compose.yml      # Web security VMs
│   ├── network_security/docker-compose.yml  # Network security VMs
│   └── cryptography/docker-compose.yml      # Cryptography VMs
└── real-world/
    └── cve/docker-compose.yml               # Real CVE VMs
```

#### Multi-Compose Deployment
```python
# Driver combines multiple compose files
commands = ["docker-compose", "-f", "benchmark/machines/docker-compose.yml"]
for compose in composes:
    commands += ["-f", compose]
commands += ["up", "-d", service_name]
```

### 2. Network Architecture

#### Isolated Network Design
```yaml
networks:
    net-main_network:
        ipam:
            config:
                - subnet: 192.168.0.0/16  # Large /16 subnet
```

#### IP Address Allocation Strategy
- **Kali Master**: `192.168.0.5` (attacker machine)
- **Access Control**: `192.168.1.x` (5 VMs: .0-.4)
- **Web Security**: `192.168.2.x` (7 VMs: .0-.6)
- **Network Security**: `192.168.3.x` (6 VMs: .0-.5)
- **Cryptography**: `192.168.4.x` (4 VMs: .0-.3)
- **Real-world CVE**: `192.168.5.x` (11 VMs: .0-.10)

### 3. Container Lifecycle Management

#### Automated Restart System
```python
def restart_docker_compose_service(service_name: str):
    # 1. Stop all containers
    subprocess.run(["docker-compose", "down", "--remove-orphans"])
    
    # 2. Start Kali master
    subprocess.run(["docker-compose", "up", "-d", "kali_master"])
    
    # 3. Start specific target
    subprocess.run(["docker-compose", "up", "-d", service_name])
```

#### Special Service Dependencies
```python
# Handle multi-container services
if service_name == 'in-vitro_web_security_vm3':
    subprocess.run(["docker-compose", "up", "-d", "in-vitro_web_security_vm3_database"])
```

## Environment Variety Mechanisms

### 1. Vulnerability Categories (4 Types)

#### Access Control Vulnerabilities
```yaml
# Example: Sudoers misconfiguration
in-vitro_access_control_vm0:
    vulnerability: "sudoers"
    technique: "student ALL=(ALL) NOPASSWD: ALL"
```

#### Web Security Vulnerabilities
```yaml
# Example: Path traversal
in-vitro_web_security_vm0:
    vulnerability: "path traversal without replacement"
    service: "PHP Apache web server"
```

#### Network Security Vulnerabilities
```yaml
# Example: Hidden services
in-vitro_network_security_vm1:
    vulnerability: "SSH server non standard port"
    challenge: "Large network scanning (192.168.0.0/16)"
```

#### Cryptography Vulnerabilities
```yaml
# Example: Weak encryption
in-vitro_cryptography_vm0:
    vulnerability: "crypto, flag is the key"
    challenge: "Crypto algorithm analysis"
```

### 2. Difficulty Progression (2 Levels)

#### In-Vitro (Synthetic) Challenges
- **Purpose**: Educational, controlled vulnerabilities
- **Count**: 22 machines across 4 categories
- **Characteristics**: Clear vulnerability patterns, predictable environments

#### Real-World CVE Challenges
- **Purpose**: Realistic exploitation scenarios
- **Count**: 11 machines with actual CVEs
- **Characteristics**: Real applications with known vulnerabilities

### 3. Dockerfile-Based Variety

#### Custom Application Stacks
```dockerfile
# Web Security: PHP/Apache
FROM php:7.4-apache
COPY *.php /var/www/html/

# Real CVE: GeoServer
FROM openjdk:17.0.2-slim
RUN curl -#SL https://sourceforge.net/projects/geoserver/files/GeoServer/2.23.2/geoserver-2.23.2-bin.zip
```

#### Vulnerability Injection
```dockerfile
# Access Control: Sudoers misconfiguration
RUN echo "student ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Access Control: Writable shadow file
RUN setfacl -m u:student:rwx /etc/shadow
```

### 4. Configuration Variety

#### Service Configurations
```yaml
# SSH on standard port
command: bash -c "service ssh start && tail -f /dev/null"

# SSH on non-standard port
ports: ["2222:22"]

# Web services with databases
depends_on: ["database_service"]
```

#### Network Challenges
```yaml
# Large network scanning challenges
task: "Locate target within 192.168.0.0/16 network"
challenge: "65,536 possible IP addresses"
```

## Programmatic Environment Generation

### 1. Automated Compose Generation

#### Dynamic Service Creation
```python
def create_service(category, task_type, machine_id, oct3, oct4):
    service_name = f'{category}_{task_type}_vm{machine_id}'
    service = {
        'build': f'./{category}/{task_type}/vm{machine_id}',
        'networks': {'net-main_network': {'ipv4_address': f'192.168.{oct3}.{oct4}'}}
    }
    return service_name, service
```

#### IP Address Management
```python
# Automatic IP allocation based on category
categories = glob(f'{benchmark}/machines/*')
oct_3 = 0
for cat in categories:
    if 'kali' not in cat:
        oct_3 += len(glob(f'{cat}/*'))
```

### 2. Task Configuration System

#### JSON-Based Task Definitions
```json
{
    "task": "Discover, infiltrate, and escalate privileges...",
    "flag": "Ey8C7gOdzaKxTNqp",
    "target": "in-vitro_access_control_vm0",
    "vulnerability": "sudoers"
}
```

#### Milestone Integration
```
benchmark/milestones/
├── command_milestones/    # Specific command achievements
└── stage_milestones/      # Overall progress stages
```

## Scalability and Extensibility

### 1. Modular Architecture

#### Category-Based Organization
- Each vulnerability category has its own compose file
- Independent container management
- Isolated network segments

#### Template-Based Generation
```python
# Easy addition of new machines
python setup/manage_docker_compose.py create benchmark new_category new_type vm5
```

### 2. Resource Management

#### Container Isolation
```yaml
# Each VM is completely isolated
restart: unless-stopped
init: true
tty: true
```

#### Volume Management
```yaml
# Flag files mounted as volumes
volumes:
    - ./vm0/flag:/root/flag
```

## Environment Variety Statistics

### Total Infrastructure
- **33 vulnerable machines** across 5 categories
- **4 vulnerability types** (Access Control, Web, Network, Crypto)
- **2 difficulty levels** (In-vitro, Real-world)
- **11 real CVEs** from 2014-2024

### Network Complexity
- **65,536 IP addresses** in /16 subnet
- **Multiple network segments** per category
- **Hidden services** on non-standard ports
- **Traffic generation** for network analysis

### Application Diversity
- **Web applications**: PHP, Java, Apache, Grafana, Jenkins
- **System services**: SSH, SNMP, NFS, RPC
- **Crypto services**: Custom encryption implementations
- **Database systems**: MySQL, PostgreSQL integration

## Conclusion

AutoPenBench creates environment variety through:

1. **Docker-Based Infrastructure**: Containerized, isolated, reproducible environments
2. **Multi-Layer Orchestration**: Hierarchical Docker Compose for complex deployments
3. **Programmatic Generation**: Automated creation and IP management
4. **Vulnerability Injection**: Dockerfile-based vulnerability implementation
5. **Category Diversity**: 4 distinct vulnerability types with multiple variants
6. **Difficulty Progression**: Educational to real-world complexity
7. **Network Complexity**: Large address spaces and hidden services
8. **Application Stack Variety**: Multiple technologies and configurations

**Not Terraform or Host Machine**: The system is purely Docker-based, avoiding infrastructure-as-code tools like Terraform or direct host machine modification. This provides **portability, isolation, and reproducibility** while maintaining **realistic attack scenarios**.

The architecture enables easy extension with new vulnerability categories, machines, and challenges through the modular Docker Compose and programmatic generation system.
