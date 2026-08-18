# 🐧 Debian Infrastructure Lab

Laboratório de infraestrutura desenvolvido com **Debian 13**, **VirtualBox**, **BIND9**, **NAT** e **rede interna**, com o objetivo de estudar e documentar conceitos de administração Linux, redes e serviços de DNS.

O projeto simula uma pequena infraestrutura com dois servidores Debian, sendo um responsável pelo **DNS primário** e outro pelo **DNS secundário**, permitindo praticar comunicação entre servidores, resolução de nomes, DNS reverso e gerenciamento de serviços.

---

## 📌 Sobre o projeto

Este laboratório foi desenvolvido como um ambiente prático de estudos para aplicação de conceitos de:

* Administração de sistemas Linux
* Virtualização
* Redes de computadores
* DNS
* BIND9
* NAT
* DNS primário e secundário
* DNS reverso
* Diagnóstico e troubleshooting

O ambiente foi construído utilizando máquinas virtuais no **VirtualBox**, permitindo simular uma infraestrutura de servidores sem a necessidade de equipamentos físicos.

---

## 🎯 Objetivos

* [x] Instalar e configurar o Debian 13
* [x] Criar máquinas virtuais utilizando VirtualBox
* [x] Configurar interfaces de rede
* [x] Implementar uma rede interna entre os servidores
* [x] Configurar acesso externo através de NAT
* [x] Instalar e configurar o BIND9
* [x] Configurar um servidor DNS primário
* [x] Configurar um servidor DNS secundário
* [x] Trabalhar com resolução DNS direta
* [x] Trabalhar com resolução DNS reversa
* [x] Validar a comunicação entre os servidores
* [x] Monitorar e analisar o funcionamento do serviço DNS
* [x] Documentar problemas e processos de troubleshooting

---

## 🏗️ Arquitetura

A infraestrutura é composta por duas máquinas virtuais Debian 13 conectadas através de uma rede interna.

```text
                         INTERNET
                            │
                            │
                     ┌──────▼──────┐
                     │  VirtualBox │
                     │     NAT     │
                     └──────┬──────┘
                            │
                       10.0.2.0/24
                            │
                     ┌──────▼──────┐
                     │     NS1     │
                     │  Debian 13  │
                     │             │
                     │    BIND9    │
                     │ DNS Primário│
                     └──────┬──────┘
                            │
                     Rede Interna
                    192.168.12.0/24
                            │
                     ┌──────▼──────┐
                     │     WEB     │
                     │  Debian 13  │
                     │             │
                     │    BIND9    │
                     │ DNS Secundário
                     └─────────────┘
```

---

## 🖥️ Servidores

### NS1 — DNS Primário

| Informação          | Configuração      |
| ------------------- | ----------------- |
| Sistema operacional | Debian 13         |
| Hostname            | `NS1`             |
| Serviço             | BIND9             |
| Interface NAT       | `enp0s3`          |
| IP NAT              | `10.0.2.15/24`    |
| Gateway             | `10.0.2.2`        |
| Interface interna   | `enp0s8`          |
| IP interno          | `192.168.12.1/24` |
| Função              | DNS Primário      |

O NS1 possui duas interfaces de rede. A primeira está conectada ao NAT do VirtualBox, permitindo comunicação externa, enquanto a segunda é utilizada para comunicação com o segundo servidor através da rede interna.

---

### WEB / NS2 — DNS Secundário

| Informação          | Configuração      |
| ------------------- | ----------------- |
| Sistema operacional | Debian 13         |
| Hostname            | `WEB`             |
| Serviço             | BIND9             |
| Interface           | `enp0s3`          |
| IP                  | `192.168.12.2/24` |
| Gateway             | `192.168.12.1`    |
| Função              | DNS Secundário    |

O segundo servidor está conectado à rede interna e utiliza o NS1 como gateway.

---

## 🌐 Configuração de rede

O laboratório utiliza duas redes distintas.

### Rede NAT

```text
10.0.2.0/24
```

Utilizada pelo NS1 para acesso externo através do VirtualBox.

```text
NS1
10.0.2.15
    │
    ▼
Gateway
10.0.2.2
    │
    ▼
VirtualBox NAT
    │
    ▼
Internet
```

### Rede interna

```text
192.168.12.0/24
```

Utilizada para comunicação entre os servidores.

```text
NS1
192.168.12.1
      │
      │
      ▼
Rede Interna
192.168.12.0/24
      │
      │
      ▼
WEB / NS2
192.168.12.2
```

---

## 🌎 DNS

O laboratório utiliza o domínio:

```text
cloudcraft.com.br
```

A infraestrutura possui dois servidores DNS:

```text
NS1 → 192.168.12.1
DNS Primário

NS2 → 192.168.12.2
DNS Secundário
```

A NS1 atua como servidor DNS primário e envia notificações para a NS2 quando ocorre uma alteração na zona.

Durante a execução do serviço, foi possível observar no log do BIND9:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Na NS2 também foi registrada a recepção da notificação:

```text
received notify for zone 'cloudcraft.com.br'
```

seguida da confirmação:

```text
zone is up to date
```

Esses registros demonstram a comunicação entre os servidores DNS.

---

## 🔄 DNS reverso

O laboratório também possui configuração relacionada à resolução reversa para a rede:

```text
192.168.12.0/24
```

A zona reversa identificada nos registros do BIND9 é:

```text
12.168.192.in-addr.arpa
```

A NS2 recebeu notificações relacionadas à zona reversa provenientes da NS1.

---

## ⚙️ BIND9

O BIND9 é o serviço responsável pelo funcionamento do DNS no laboratório.

Na NS1, o serviço encontra-se configurado como:

```text
named.service
Active: active (running)
Loaded: loaded
enabled
```

Na NS2, o serviço também está configurado como:

```text
named.service
Active: active (running)
Loaded: loaded
enabled
```

O processo utilizado pelo serviço é:

```text
/usr/sbin/named -f -u bind
```

---

## 🧪 Testes e validação

Durante a configuração do laboratório, foram utilizados comandos de administração e diagnóstico do Linux e do BIND9.

### Verificação das interfaces

```bash
ip addr
```

### Verificação das rotas

```bash
ip route
```

### Verificação do serviço DNS

```bash
sudo systemctl status bind9
```

### Validação da configuração do BIND

```bash
sudo named-checkconf
```

### Validação de zonas

```bash
sudo named-checkzone
```

### Testes de conectividade

```bash
ping
```

### Consultas DNS

```bash
dig
```

```bash
nslookup
```

---

## 🛠️ Troubleshooting

Durante a implementação foram encontrados comportamentos e mensagens de erro relacionados à resolução DNS e comunicação externa.

Entre os registros observados na NS2 estavam mensagens relacionadas a:

```text
broken trust chain resolving
```

e:

```text
resolver priming query complete: timed out
```

Esses eventos foram analisados juntamente com os registros posteriores do BIND9.

Apesar dessas mensagens, o serviço permaneceu em execução e posteriormente foram registradas comunicações bem-sucedidas entre NS1 e NS2, incluindo notificações das zonas DNS.

Esta seção será expandida conforme novos testes e análises forem realizados no laboratório.

---

## 🧰 Tecnologias utilizadas

| Tecnologia          | Utilização                         |
| ------------------- | ---------------------------------- |
| 🐧 Debian 13        | Sistema operacional dos servidores |
| 📦 VirtualBox       | Plataforma de virtualização        |
| 🌐 BIND9            | Serviço DNS                        |
| 🔀 NAT              | Acesso externo da NS1              |
| 🔗 Internal Network | Comunicação entre os servidores    |
| 🖥️ Linux CLI       | Administração e diagnóstico        |
| 🌎 DNS              | Resolução de nomes                 |
| 🔄 DNS Reverso      | Resolução de endereços IP          |

---

## 📚 Documentação

A documentação detalhada do laboratório está organizada na pasta `docs/`.

Em desenvolvimento:

* [Arquitetura](docs/arquitetura.md)
* [Configuração de rede](docs/rede.md)
* [Configuração DNS](docs/dns.md)
* [Servidor NS1](docs/ns1.md)
* [Servidor NS2](docs/ns2.md)
* [Testes e validação](docs/testes.md)
* [Troubleshooting](docs/troubleshooting.md)

---

## 🚀 Próximos passos

Como evolução do laboratório, podem ser implementados:

* [ ] Documentação completa das configurações do BIND9
* [ ] Testes detalhados de resolução direta
* [ ] Testes detalhados de resolução reversa
* [ ] Validação da sincronização entre DNS primário e secundário
* [ ] Implementação de servidor Web
* [ ] Configuração de HTTPS
* [ ] Firewall utilizando `nftables`
* [ ] Monitoramento dos serviços
* [ ] Centralização de logs
* [ ] Hardening dos servidores
* [ ] Criação de novos serviços para ampliar o laboratório

---

## 👨‍💻 Autor

**Gabriel Lins Lamoréa**

Estudante de Segurança da Informação | Infraestrutura | Linux | Redes | Cloud

Este projeto faz parte do meu portfólio acadêmico e profissional, com foco no desenvolvimento prático de conhecimentos em infraestrutura, administração Linux, redes e serviços de TI.

---

## 📄 Licença

Este projeto está disponível para fins educacionais e de portfólio.
