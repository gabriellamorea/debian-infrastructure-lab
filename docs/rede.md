# 🌐 Configuração de Rede

## 1. Visão geral

A infraestrutura utiliza duas redes distintas dentro do ambiente virtualizado:

* **Rede NAT:** utilizada pela NS1 para acesso externo.
* **Rede interna:** utilizada para comunicação entre a NS1 e a WEB/NS2.

Essa configuração permite simular uma infraestrutura onde um servidor possui conectividade externa e também funciona como ponto de comunicação com uma rede interna.

---

## 2. Topologia de rede

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
                     └──────┬──────┘
                            │
                    192.168.12.1
                            │
                     ───────┼───────
                    Rede Interna
                     192.168.12.0/24
                            │
                    192.168.12.2
                     ┌──────▼──────┐
                     │     WEB     │
                     │  Debian 13  │
                     └─────────────┘
```

---

# 3. NS1

A NS1 possui duas interfaces de rede:

```text
enp0s3
enp0s8
```

Essa configuração permite que a máquina participe simultaneamente da rede NAT e da rede interna.

---

## 3.1 Interface enp0s3

A interface `enp0s3` está conectada ao NAT do VirtualBox.

Configuração identificada:

| Parâmetro | Valor          |
| --------- | -------------- |
| Interface | `enp0s3`       |
| IPv4      | `10.0.2.15/24` |
| Rede      | `10.0.2.0/24`  |
| Broadcast | `10.0.2.255`   |
| Gateway   | `10.0.2.2`     |
| Método    | DHCP           |

Essa interface é responsável pela conectividade externa da NS1.

O endereço `10.0.2.15` foi atribuído dinamicamente pelo serviço DHCP do NAT do VirtualBox.

---

## 3.2 Interface enp0s8

A interface `enp0s8` está conectada à rede interna.

Configuração:

| Parâmetro | Valor               |
| --------- | ------------------- |
| Interface | `enp0s8`            |
| IPv4      | `192.168.12.1/24`   |
| Rede      | `192.168.12.0/24`   |
| Broadcast | `192.168.12.255`    |
| Gateway   | Não configurado     |
| Função    | Comunicação interna |

O endereço `192.168.12.1` funciona como endereço da NS1 dentro da rede interna.

Essa interface é utilizada para comunicação com a WEB/NS2.

---

# 4. Tabela de interfaces da NS1

| Interface | IP                | Rede              | Função               |
| --------- | ----------------- | ----------------- | -------------------- |
| `lo`      | `127.0.0.1/8`     | Loopback          | Comunicação local    |
| `enp0s3`  | `10.0.2.15/24`    | `10.0.2.0/24`     | NAT / acesso externo |
| `enp0s8`  | `192.168.12.1/24` | `192.168.12.0/24` | Rede interna         |

---

# 5. Tabela de rotas da NS1

A tabela de roteamento identificada foi:

```text
default via 10.0.2.2 dev enp0s3
10.0.2.0/24 dev enp0s3
192.168.12.0/24 dev enp0s8
```

### Rota padrão

```text
default via 10.0.2.2 dev enp0s3
```

Indica que o tráfego destinado a redes que não possuem uma rota específica deve ser encaminhado para o gateway `10.0.2.2` através da interface `enp0s3`.

### Rede NAT

```text
10.0.2.0/24 dev enp0s3
```

Indica que a NS1 está diretamente conectada à rede `10.0.2.0/24`.

### Rede interna

```text
192.168.12.0/24 dev enp0s8
```

Indica que a rede `192.168.12.0/24` está diretamente conectada à interface `enp0s8`.

---

# 6. WEB / NS2

A WEB possui uma interface de rede utilizada para comunicação com a rede interna.

## 6.1 Interface enp0s3

Configuração identificada:

| Parâmetro | Valor             |
| --------- | ----------------- |
| Interface | `enp0s3`          |
| IPv4      | `192.168.12.2/24` |
| Rede      | `192.168.12.0/24` |
| Broadcast | `192.168.12.255`  |
| Gateway   | `192.168.12.1`    |
| Função    | Rede interna      |

A interface utiliza o endereço `192.168.12.2`.

O gateway configurado é a própria NS1:

```text
192.168.12.1
```

Dessa forma, a NS1 funciona como ponto de saída da WEB/NS2 para outras redes.

---

# 7. Tabela de interfaces da WEB

| Interface | IP                | Rede              | Função              |
| --------- | ----------------- | ----------------- | ------------------- |
| `lo`      | `127.0.0.1/8`     | Loopback          | Comunicação local   |
| `enp0s3`  | `192.168.12.2/24` | `192.168.12.0/24` | Comunicação interna |

---

# 8. Tabela de rotas da WEB

A tabela de rotas identificada foi:

```text
default via 192.168.12.1 dev enp0s3 onlink
192.168.12.0/24 dev enp0s3
```

A rota padrão direciona o tráfego para:

```text
192.168.12.1
```

que corresponde ao endereço interno da NS1.

A rota da rede:

```text
192.168.12.0/24
```

indica que a WEB possui conexão direta com os dispositivos dessa rede através da interface `enp0s3`.

---

# 9. Comunicação entre os servidores

A comunicação entre NS1 e WEB ocorre através da rede:

```text
192.168.12.0/24
```

Com os seguintes endereços:

```text
NS1 → 192.168.12.1
WEB → 192.168.12.2
```

O fluxo básico é:

```text
NS1
192.168.12.1
      │
      │
      │ Rede interna
      │
      ▼
WEB
192.168.12.2
```

Essa rede também é utilizada para a comunicação relacionada ao serviço DNS.

---

# 10. Relação entre NAT e rede interna

A NS1 possui acesso às duas redes:

```text
             NS1
              │
       ┌──────┴──────┐
       │             │
       ▼             ▼
   enp0s3          enp0s8
10.0.2.15       192.168.12.1
       │             │
       ▼             ▼
      NAT      Rede Interna
       │             │
   Internet       WEB/NS2
                  192.168.12.2
```

Essa configuração cria uma separação lógica entre a rede externa e a rede interna do laboratório.

---

# 11. Comandos utilizados

Durante a configuração e diagnóstico da rede, o principal comando utilizado foi:

```bash
ip addr
```

Utilizado para visualizar:

* Interfaces disponíveis;
* Endereços IP;
* Máscaras;
* Estado das interfaces;
* Endereços IPv6;
* Endereços MAC.

Para visualizar as rotas:

```bash
ip route
```

Esse comando permite verificar:

* Gateway padrão;
* Redes diretamente conectadas;
* Interface utilizada para cada rota;
* Endereço de origem utilizado pelo sistema.

---

# 12. Conceitos praticados

A configuração deste laboratório permitiu praticar os seguintes conceitos:

* IPv4
* Máscara de rede
* Gateway
* Roteamento
* DHCP
* NAT
* Rede interna
* Interfaces de rede
* Loopback
* Endereçamento privado
* Comunicação entre servidores
* Segmentação de redes

---

## 13. Resumo

A configuração de rede do laboratório utiliza a NS1 como ponto central da infraestrutura.

A NS1 possui uma interface conectada ao NAT do VirtualBox e outra conectada à rede interna. A WEB/NS2 utiliza a rede interna para comunicação com a NS1.

Essa arquitetura fornece uma base para os serviços de DNS implementados posteriormente no laboratório.
