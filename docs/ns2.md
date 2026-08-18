# 🖥️ Servidor NS2 / WEB

## 1. Visão geral

A máquina **WEB** atua como segundo servidor DNS da infraestrutura, sendo identificada no contexto do serviço DNS como **NS2**.

O servidor executa **Debian 13** e está conectado à rede interna do laboratório, utilizando a NS1 como gateway.

O principal serviço executado nessa máquina é o **BIND9**, configurado para atuar como servidor DNS secundário.

---

# 2. Identificação

| Informação          | Valor             |
| ------------------- | ----------------- |
| Hostname observado  | `WEB`             |
| Identificação DNS   | `NS2`             |
| Sistema operacional | Debian 13         |
| Serviço             | BIND9             |
| Função              | DNS Secundário    |
| Rede                | `192.168.12.0/24` |
| Endereço IP         | `192.168.12.2/24` |
| Gateway             | `192.168.12.1`    |

> **Observação:** o hostname do sistema é `WEB`, enquanto `NS2` representa sua função dentro da arquitetura DNS do laboratório.

---

# 3. Interface de rede

A WEB possui uma interface Ethernet utilizada para comunicação com a rede interna:

```text
enp0s3
```

Também está presente a interface de loopback:

```text
lo
```

---

## 3.1 Loopback

A interface:

```text
lo
```

possui o endereço:

```text
127.0.0.1/8
```

Também foi identificado o endereço IPv6 de loopback:

```text
::1/128
```

A interface de loopback permite a comunicação interna do próprio sistema operacional.

---

## 3.2 Interface enp0s3

A interface `enp0s3` está conectada à rede interna do laboratório.

Configuração identificada:

| Parâmetro | Valor               |
| --------- | ------------------- |
| Interface | `enp0s3`            |
| IPv4      | `192.168.12.2/24`   |
| Rede      | `192.168.12.0/24`   |
| Broadcast | `192.168.12.255`    |
| Gateway   | `192.168.12.1`      |
| Função    | Comunicação interna |

Essa interface permite a comunicação direta com a NS1.

---

# 4. Endereçamento

A configuração de rede da WEB pode ser representada:

```text
┌──────────────────────────────────┐
│              WEB / NS2           │
├──────────────────────────────────┤
│ enp0s3                           │
│ IP: 192.168.12.2/24              │
│ Gateway: 192.168.12.1            │
│ Rede: 192.168.12.0/24            │
└──────────────────────────────────┘
```

A máquina está localizada exclusivamente na rede interna utilizada pelo laboratório.

---

# 5. Tabela de roteamento

A tabela de rotas identificada na WEB foi:

```text
default via 192.168.12.1 dev enp0s3 onlink
192.168.12.0/24 dev enp0s3
```

## 5.1 Rota padrão

```text
default via 192.168.12.1 dev enp0s3 onlink
```

A rota padrão direciona o tráfego destinado a redes externas para:

```text
192.168.12.1
```

Esse endereço pertence à NS1.

---

## 5.2 Rede diretamente conectada

```text
192.168.12.0/24 dev enp0s3
```

Essa rota indica que a WEB possui conexão direta com a rede interna `192.168.12.0/24`.

---

# 6. Relação com a NS1

A comunicação entre os dois servidores ocorre através da rede:

```text
192.168.12.0/24
```

Com os seguintes endereços:

```text
NS1 → 192.168.12.1
NS2 → 192.168.12.2
```

A topologia é:

```text
┌──────────────┐
│     NS1      │
│ 192.168.12.1 │
└──────┬───────┘
       │
       │ Rede Interna
       │ 192.168.12.0/24
       │
┌──────▼───────┐
│  WEB / NS2   │
│ 192.168.12.2 │
└──────────────┘
```

Essa comunicação é fundamental para o funcionamento da infraestrutura DNS.

---

# 7. Serviço BIND9

A WEB executa o serviço BIND9.

O serviço é apresentado pelo systemd como:

```text
named.service
```

O processo em execução é:

```text
/usr/sbin/named -f -u bind
```

---

## 7.1 Estado do serviço

Durante a verificação, o serviço apresentou:

```text
Loaded: loaded
Active: active (running)
```

O serviço também estava configurado como:

```text
enabled
```

Portanto, o BIND9 encontra-se ativo e configurado para iniciar automaticamente junto com o sistema.

---

# 8. Função DNS

No laboratório, a WEB atua como **servidor DNS secundário**.

Ela mantém as informações recebidas do servidor DNS primário e participa do processo de atualização das zonas.

A principal zona observada foi:

```text
cloudcraft.com.br
```

Também foi observada comunicação relacionada à zona reversa:

```text
12.168.192.in-addr.arpa
```

---

# 9. Recebimento de DNS NOTIFY

Um dos principais registros encontrados nos logs da WEB demonstra o recebimento de uma notificação da NS1:

```text
received notify for zone 'cloudcraft.com.br'
```

A origem da comunicação foi:

```text
192.168.12.1
```

que corresponde ao endereço interno da NS1.

Esse mecanismo permite que o servidor secundário seja informado sobre alterações na zona mantida pelo servidor primário.

---

# 10. Validação da zona

Após receber a notificação da NS1, o BIND9 registrou:

```text
zone cloudcraft.com.br/IN:
notify from 192.168.12.1:
zone is up to date
```

Essa mensagem indica que, no momento registrado, a zona `cloudcraft.com.br` estava atualizada na NS2.

Esse é um dos principais indícios encontrados durante o laboratório de que a comunicação entre os servidores DNS estava funcionando conforme esperado.

---

# 11. DNS reverso

A WEB também recebeu uma notificação relacionada à zona reversa:

```text
12.168.192.in-addr.arpa
```

O registro observado foi:

```text
received notify for zone '12.168.192.in-addr.arpa'
```

A notificação foi originada pelo endereço:

```text
192.168.12.1
```

Esse comportamento demonstra que a comunicação entre o DNS primário e secundário também envolve a zona reversa.

---

# 12. Logs do BIND9

Os logs foram utilizados para verificar o comportamento do serviço durante o laboratório.

Entre os eventos identificados estão:

* Inicialização do BIND9;
* Consultas de resolução;
* Eventos relacionados à resolução externa;
* Recebimento de notificações DNS;
* Atualização de zonas;
* Comunicação entre NS1 e NS2.

Alguns registros relacionados à resolução externa apresentaram mensagens como:

```text
broken trust chain resolving
```

e:

```text
resolver priming query complete: timed out
```

Esses eventos foram registrados durante a execução do serviço, porém o BIND9 permaneceu ativo e posteriormente apresentou comunicação bem-sucedida com a NS1.

---

# 13. Administração do servidor

Os principais comandos utilizados para diagnóstico da WEB/NS2 incluem:

### Verificar interfaces

```bash
ip addr
```

### Verificar rotas

```bash
ip route
```

### Verificar o BIND9

```bash
sudo systemctl status bind9
```

### Verificar configuração

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

### Realizar consultas DNS

```bash
nslookup
```

---

# 14. Fluxo DNS

O funcionamento da NS2 dentro da infraestrutura pode ser representado:

```text
                    NS1
               192.168.12.1
                     │
                     │
                  NOTIFY
                     │
                     ▼
              ┌─────────────┐
              │  WEB / NS2  │
              │192.168.12.2 │
              │             │
              │   BIND9     │
              │DNS Secundário
              └──────┬──────┘
                     │
                     ▼
             Verificação da zona
                     │
                     ▼
                Zone is up
                   to date
```

---

# 15. Checklist operacional

* [x] Debian 13 instalado
* [x] Hostname identificado como `WEB`
* [x] Servidor utilizado como `NS2`
* [x] Interface `enp0s3` configurada
* [x] Endereço `192.168.12.2/24` configurado
* [x] Gateway `192.168.12.1` configurado
* [x] Conectividade com a rede interna
* [x] BIND9 instalado
* [x] BIND9 ativo
* [x] BIND9 habilitado no boot
* [x] Zona `cloudcraft.com.br` identificada
* [x] Zona reversa identificada
* [x] DNS NOTIFY recebido da NS1
* [x] Zona reportada como atualizada

---

# 16. Conclusão

A WEB/NS2 desempenha o papel de servidor DNS secundário no laboratório.

Conectada à rede interna `192.168.12.0/24`, a máquina utiliza o endereço `192.168.12.2` e a NS1 (`192.168.12.1`) como gateway.

O BIND9 encontra-se ativo e habilitado, permitindo que a máquina participe da infraestrutura DNS.

Os registros analisados demonstraram o recebimento de notificações DNS provenientes da NS1 para as zonas `cloudcraft.com.br` e `12.168.192.in-addr.arpa`.

A confirmação de que a zona `cloudcraft.com.br` estava `up to date` representa uma evidência importante do funcionamento da comunicação entre o servidor DNS primário e o secundário.
