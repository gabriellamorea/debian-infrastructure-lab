# 🖥️ Servidor NS1

## 1. Visão geral

A **NS1** é o servidor principal do laboratório de infraestrutura.

A máquina executa **Debian 13** e possui duas interfaces de rede, permitindo sua participação simultânea na rede NAT do VirtualBox e na rede interna utilizada para comunicação com o segundo servidor.

Além da função de conectividade entre as redes, a NS1 executa o serviço **BIND9**, atuando como servidor DNS primário do laboratório.

---

## 2. Identificação

| Informação          | Valor             |
| ------------------- | ----------------- |
| Hostname            | `NS1`             |
| Sistema operacional | Debian 13         |
| Serviço DNS         | BIND9             |
| Função principal    | DNS Primário      |
| Rede NAT            | `10.0.2.0/24`     |
| Rede interna        | `192.168.12.0/24` |

---

# 3. Interfaces de rede

A NS1 possui duas interfaces Ethernet:

```text
enp0s3
enp0s8
```

Além da interface de loopback:

```text
lo
```

---

## 3.1 Loopback

A interface:

```text
lo
```

utiliza:

```text
127.0.0.1/8
```

A interface de loopback é utilizada para comunicação interna do próprio sistema operacional.

Também foi identificado o endereço IPv6:

```text
::1/128
```

---

## 3.2 Interface enp0s3

A interface `enp0s3` está conectada à rede NAT do VirtualBox.

Configuração identificada:

| Parâmetro            | Valor                 |
| -------------------- | --------------------- |
| Interface            | `enp0s3`              |
| IPv4                 | `10.0.2.15/24`        |
| Rede                 | `10.0.2.0/24`         |
| Broadcast            | `10.0.2.255`          |
| Gateway              | `10.0.2.2`            |
| Método de atribuição | DHCP                  |
| Função               | Conectividade externa |

O endereço IP foi atribuído dinamicamente através do DHCP disponibilizado pelo NAT do VirtualBox.

---

## 3.3 Interface enp0s8

A interface `enp0s8` está conectada à rede interna.

Configuração:

| Parâmetro | Valor               |
| --------- | ------------------- |
| Interface | `enp0s8`            |
| IPv4      | `192.168.12.1/24`   |
| Rede      | `192.168.12.0/24`   |
| Broadcast | `192.168.12.255`    |
| Função    | Comunicação interna |

Essa interface é responsável pela comunicação direta com a WEB/NS2.

---

# 4. Endereçamento

A configuração IPv4 da NS1 pode ser resumida:

```text
┌──────────────────────────────────┐
│              NS1                 │
├──────────────────────────────────┤
│ enp0s3                           │
│ IP: 10.0.2.15/24                 │
│ Gateway: 10.0.2.2                │
│ Rede: 10.0.2.0/24                │
├──────────────────────────────────┤
│ enp0s8                           │
│ IP: 192.168.12.1/24              │
│ Rede: 192.168.12.0/24            │
└──────────────────────────────────┘
```

---

# 5. Tabela de roteamento

A tabela de rotas observada na NS1 foi:

```text
default via 10.0.2.2 dev enp0s3
10.0.2.0/24 dev enp0s3
192.168.12.0/24 dev enp0s8
```

## 5.1 Rota padrão

```text
default via 10.0.2.2 dev enp0s3
```

A rota padrão determina que o tráfego destinado a redes que não possuem uma rota específica seja encaminhado para o gateway `10.0.2.2`.

A interface utilizada é:

```text
enp0s3
```

---

## 5.2 Rede NAT

```text
10.0.2.0/24 dev enp0s3
```

Indica que a NS1 está diretamente conectada à rede `10.0.2.0/24`.

---

## 5.3 Rede interna

```text
192.168.12.0/24 dev enp0s8
```

Indica que a rede interna está diretamente conectada à interface `enp0s8`.

---

# 6. Serviço BIND9

A NS1 executa o serviço BIND9, utilizado como servidor DNS primário.

O serviço é identificado pelo systemd como:

```text
named.service
```

O processo executado é:

```text
/usr/sbin/named -f -u bind
```

---

## 6.1 Estado do serviço

Durante a verificação do servidor, o serviço apresentou:

```text
Loaded: loaded
Active: active (running)
```

O serviço também estava configurado como:

```text
enabled
```

Isso significa que o BIND9 está ativo e configurado para inicialização automática com o sistema.

---

# 7. Função DNS

A NS1 atua como **DNS Primário** da infraestrutura.

Uma das zonas configuradas é:

```text
cloudcraft.com.br
```

A NS1 mantém as informações da zona e comunica atualizações ao servidor secundário.

---

# 8. Comunicação com a NS2

A comunicação DNS entre os servidores ocorre através da rede interna:

```text
192.168.12.0/24
```

Com os endereços:

```text
NS1 → 192.168.12.1
NS2 → 192.168.12.2
```

A NS1 envia notificações DNS para a NS2.

Durante a execução do serviço, foi registrado:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Esse registro demonstra que a NS1 enviou uma notificação DNS para o servidor secundário.

---

# 9. DNS reverso

Além da zona direta, a infraestrutura possui uma zona reversa associada à rede interna.

A zona identificada é:

```text
12.168.192.in-addr.arpa
```

Essa configuração permite trabalhar com resolução reversa de endereços IPv4 da rede:

```text
192.168.12.0/24
```

---

# 10. Administração do servidor

Alguns dos principais comandos utilizados para administrar e diagnosticar a NS1 são:

### Verificar interfaces

```bash
ip addr
```

### Verificar rotas

```bash
ip route
```

### Verificar o serviço DNS

```bash
sudo systemctl status bind9
```

### Verificar configuração do BIND

```bash
sudo named-checkconf
```

### Verificar zonas

```bash
sudo named-checkzone
```

### Consultar DNS

```bash
dig
```

### Consultar DNS através do nslookup

```bash
nslookup
```

---

# 11. Análise de logs

Os logs do BIND9 foram utilizados para acompanhar o comportamento do serviço.

Entre os eventos identificados estão:

* Inicialização do serviço;
* Ativação das interfaces de escuta;
* Notificações DNS;
* Comunicação com o servidor secundário;
* Atualização de zonas;
* Eventos relacionados à resolução DNS externa.

Um dos registros mais relevantes foi:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Esse evento confirma que a NS1 está realizando a comunicação esperada com o servidor DNS secundário.

---

# 12. Modelo de funcionamento

A função da NS1 dentro do laboratório pode ser resumida:

```text
                       INTERNET
                           │
                           ▼
                     VirtualBox NAT
                           │
                     10.0.2.0/24
                           │
                    ┌──────▼──────┐
                    │     NS1     │
                    │  Debian 13  │
                    │             │
                    │   BIND9     │
                    │ DNS Primário│
                    └──────┬──────┘
                           │
                    192.168.12.0/24
                           │
                           ▼
                         NS2
                    192.168.12.2
```

A NS1 funciona como ponto central da infraestrutura, conectando a rede NAT à rede interna e fornecendo o serviço DNS primário.

---

# 13. Checklist operacional

* [x] Debian 13 instalado
* [x] Hostname configurado como `NS1`
* [x] Interface NAT configurada
* [x] Interface interna configurada
* [x] Endereço `10.0.2.15/24` identificado
* [x] Endereço `192.168.12.1/24` identificado
* [x] Gateway `10.0.2.2` configurado
* [x] BIND9 instalado
* [x] BIND9 ativo
* [x] BIND9 habilitado no boot
* [x] Zona `cloudcraft.com.br` configurada
* [x] Comunicação com a NS2 observada
* [x] DNS reverso identificado

---

# 14. Conclusão

A NS1 representa o principal componente da infraestrutura do laboratório.

A combinação de duas interfaces de rede permite que o servidor tenha acesso à rede externa através do NAT e, simultaneamente, mantenha comunicação com a rede interna.

A instalação e execução do BIND9 transformam a NS1 no servidor DNS primário da infraestrutura, responsável pela manutenção das zonas DNS e pela comunicação com o servidor secundário.

A configuração fornece uma base prática para estudos posteriores de administração Linux, DNS, segurança de servidores, roteamento, monitoramento e outros serviços de infraestrutura.
