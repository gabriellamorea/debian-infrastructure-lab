# Arquitetura do Laboratório

## 1. Visão geral

O laboratório foi desenvolvido utilizando máquinas virtuais Debian 13 executadas através do VirtualBox.

O objetivo da arquitetura é simular uma pequena infraestrutura de servidores, permitindo a prática de conceitos relacionados a virtualização, redes, Linux e serviços DNS.

A infraestrutura é composta por duas máquinas virtuais:

* **NS1** — servidor DNS primário
* **WEB** — servidor DNS secundário

As máquinas são conectadas através de uma rede interna, enquanto a NS1 possui uma segunda interface conectada à rede NAT do VirtualBox.

---

## 2. Diagrama da infraestrutura

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
                      10.0.2.15
                     ┌──────▼──────┐
                     │     NS1     │
                     │  Debian 13  │
                     │             │
                     │    BIND9    │
                     │ DNS Primário│
                     └──────┬──────┘
                            │
                     192.168.12.0/24
                            │
                      192.168.12.2
                     ┌──────▼──────┐
                     │     WEB     │
                     │  Debian 13  │
                     │             │
                     │    BIND9    │
                     │ DNS Secundário
                     └─────────────┘
```

---

## 3. Componentes

### 3.1 NS1

A NS1 é o servidor principal do laboratório.

Possui duas interfaces de rede:

| Interface | Endereço          | Função       |
| --------- | ----------------- | ------------ |
| `enp0s3`  | `10.0.2.15/24`    | NAT          |
| `enp0s8`  | `192.168.12.1/24` | Rede interna |

A interface `enp0s3` possui como gateway:

```text
10.0.2.2
```

Essa configuração permite que a NS1 tenha acesso à rede externa através do NAT fornecido pelo VirtualBox.

A interface `enp0s8` é utilizada para comunicação com o segundo servidor.

---

### 3.2 WEB / NS2

A segunda máquina possui uma única interface utilizada na rede interna:

| Interface | Endereço          | Função       |
| --------- | ----------------- | ------------ |
| `enp0s3`  | `192.168.12.2/24` | Rede interna |

O gateway configurado é:

```text
192.168.12.1
```

Dessa forma, a NS1 atua como ponto de saída da rede interna.

---

## 4. Segmentação da rede

A infraestrutura utiliza duas redes.

### NAT

```text
10.0.2.0/24
```

Responsável pela conectividade externa da NS1.

### Rede interna

```text
192.168.12.0/24
```

Responsável pela comunicação entre NS1 e WEB.

A separação permite simular uma arquitetura onde o servidor principal possui conectividade externa e uma interface dedicada à comunicação com servidores internos.

---

## 5. Fluxo de comunicação

O fluxo básico da infraestrutura pode ser representado da seguinte maneira:

```text
Internet
   │
   ▼
VirtualBox NAT
   │
   ▼
NS1
10.0.2.15
   │
   │ 192.168.12.1
   ▼
Rede Interna
192.168.12.0/24
   │
   ▼
WEB / NS2
192.168.12.2
```

A NS1 possui conectividade com as duas redes, enquanto a WEB/NS2 está conectada à rede interna.

---

## 6. Comunicação DNS

Além da comunicação de rede, os servidores mantêm comunicação relacionada ao serviço DNS.

A NS1 atua como servidor DNS primário e a WEB/NS2 como servidor DNS secundário.

```text
NS1
192.168.12.1
   │
   │ DNS NOTIFY
   ▼
NS2
192.168.12.2
```

Durante a execução do BIND9, foram observados registros indicando que a NS1 enviou notificações para a NS2 relacionadas à zona:

```text
cloudcraft.com.br
```

A NS2 registrou o recebimento da notificação e indicou que a zona estava atualizada.

---

## 7. Serviços

Os principais serviços presentes no laboratório são:

| Serviço      | Servidor         | Função                       |
| ------------ | ---------------- | ---------------------------- |
| BIND9        | NS1              | DNS Primário                 |
| BIND9        | WEB/NS2          | DNS Secundário               |
| NAT          | NS1 / VirtualBox | Conectividade externa        |
| Rede interna | NS1 + NS2        | Comunicação entre servidores |

---

## 8. Objetivo da arquitetura

A arquitetura foi criada para fornecer um ambiente controlado de estudos, permitindo experimentar configurações de infraestrutura sem interferir em ambientes produtivos.

O laboratório também serve como base para futuras implementações, como:

* Servidor Web
* HTTPS
* Firewall
* Monitoramento
* Centralização de logs
* Hardening
* Novos serviços de infraestrutura
