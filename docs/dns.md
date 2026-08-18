# 🌎 DNS e BIND9

## 1. Visão geral

O laboratório utiliza o **BIND9** como serviço de DNS (Domain Name System).

O DNS é responsável por realizar a resolução de nomes, permitindo que dispositivos e aplicações utilizem nomes de domínio em vez de depender diretamente de endereços IP.

Neste laboratório, foi implementada uma estrutura composta por:

* **NS1** — servidor DNS primário;
* **WEB/NS2** — servidor DNS secundário;
* Zona direta `cloudcraft.com.br`;
* Zona reversa para a rede `192.168.12.0/24`;
* Comunicação entre os servidores através da rede interna;
* Notificações DNS entre o servidor primário e o secundário.

---

# 2. Arquitetura DNS

A estrutura DNS pode ser representada da seguinte forma:

```text
                    DNS
                     │
                     │
              ┌──────▼──────┐
              │     NS1     │
              │ 192.168.12.1│
              │             │
              │ DNS Primário│
              │   BIND9     │
              └──────┬──────┘
                     │
                 NOTIFY
                     │
                     ▼
              ┌──────────────┐
              │     WEB      │
              │ 192.168.12.2 │
              │              │
              │ DNS Secundário
              │    BIND9     │
              └──────────────┘
```

A NS1 mantém a zona principal e informa a NS2 quando alterações são detectadas.

---

# 3. Servidor DNS Primário

O servidor **NS1** atua como DNS primário do laboratório.

Suas principais informações são:

| Informação | Valor          |
| ---------- | -------------- |
| Hostname   | `NS1`          |
| Sistema    | Debian 13      |
| Serviço    | BIND9          |
| IP interno | `192.168.12.1` |
| Função     | DNS Primário   |

O serviço utilizado pelo BIND9 é:

```text
named.service
```

O processo executado é:

```text
/usr/sbin/named -f -u bind
```

---

# 4. Servidor DNS Secundário

A máquina **WEB**, utilizada como NS2 no contexto DNS, atua como servidor secundário.

| Informação | Valor          |
| ---------- | -------------- |
| Hostname   | `WEB`          |
| Sistema    | Debian 13      |
| Serviço    | BIND9          |
| IP interno | `192.168.12.2` |
| Função     | DNS Secundário |

A comunicação entre os servidores ocorre através da rede:

```text
192.168.12.0/24
```

---

# 5. Zona direta

O domínio utilizado no laboratório é:

```text
cloudcraft.com.br
```

A zona direta é responsável pela resolução de nomes para endereços IP.

A estrutura pode ser representada como:

```text
cloudcraft.com.br
        │
        ├── NS1 → 192.168.12.1
        │
        └── NS2 → 192.168.12.2
```

O servidor primário mantém os dados da zona e o servidor secundário mantém uma cópia sincronizada.

---

# 6. DNS NOTIFY

Uma das funcionalidades observadas durante os testes foi o mecanismo **DNS NOTIFY**.

Quando uma alteração é realizada na zona do servidor primário, o BIND9 pode enviar uma notificação para os servidores secundários informando que existe uma nova versão da zona.

No laboratório, a NS1 registrou:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Esse registro demonstra que a NS1 enviou uma notificação DNS para o endereço:

```text
192.168.12.2
```

que corresponde à WEB/NS2.

---

# 7. Recebimento da notificação na NS2

Na NS2 foi observado o recebimento da notificação:

```text
received notify for zone 'cloudcraft.com.br'
```

Posteriormente, o BIND9 registrou:

```text
zone cloudcraft.com.br/IN:
notify from 192.168.12.1:
zone is up to date
```

Esse comportamento demonstra a comunicação entre o servidor primário e o secundário.

A NS2 recebeu a informação enviada pela NS1 e verificou o estado da zona.

---

# 8. DNS reverso

Além da zona direta, o laboratório possui uma configuração relacionada ao DNS reverso.

A rede interna utilizada é:

```text
192.168.12.0/24
```

Para IPv4, a zona reversa correspondente utiliza a representação:

```text
12.168.192.in-addr.arpa
```

Essa zona é utilizada para resolver endereços IP de volta para nomes DNS.

### Resolução direta

```text
nome → IP
```

Exemplo conceitual:

```text
servidor.cloudcraft.com.br
        ↓
192.168.12.x
```

### Resolução reversa

```text
IP → nome
```

Exemplo conceitual:

```text
192.168.12.x
        ↓
servidor.cloudcraft.com.br
```

---

# 9. Notificação da zona reversa

A NS2 também registrou uma notificação relacionada à zona reversa:

```text
received notify for zone '12.168.192.in-addr.arpa'
```

A notificação foi recebida a partir do endereço:

```text
192.168.12.1
```

Esse comportamento demonstra que a comunicação DNS entre os servidores não está limitada apenas à zona direta.

---

# 10. Estado do BIND9

O serviço BIND9 encontra-se ativo nas duas máquinas.

### NS1

```text
named.service
Active: active (running)
```

O serviço também está habilitado:

```text
enabled
```

Isso permite que o serviço seja iniciado automaticamente durante a inicialização do sistema.

### WEB / NS2

A mesma condição foi observada na segunda máquina:

```text
named.service
Active: active (running)
enabled
```

---

# 11. Funcionamento simplificado

O funcionamento do laboratório pode ser resumido da seguinte forma:

```text
                    ALTERAÇÃO NA ZONA
                           │
                           ▼
                    ┌─────────────┐
                    │     NS1     │
                    │ DNS Primário│
                    └──────┬──────┘
                           │
                           │ NOTIFY
                           ▼
                    ┌─────────────┐
                    │     NS2     │
                    │DNS Secundário
                    └──────┬──────┘
                           │
                           ▼
                    Verificação da
                         zona
                           │
                    ┌──────▼──────┐
                    │ Zona atualizada
                    └─────────────┘
```

Esse modelo reduz a necessidade de configuração manual da zona secundária sempre que ocorre uma atualização.

---

# 12. Diagnóstico do serviço

Durante o laboratório, foram utilizados comandos para verificar o funcionamento do BIND9.

### Status do serviço

```bash
sudo systemctl status bind9
```

Dependendo da distribuição e configuração do sistema, o serviço pode ser apresentado pelo nome:

```text
named.service
```

### Verificação da configuração

```bash
sudo named-checkconf
```

Esse comando permite verificar a configuração do BIND9 e identificar erros de sintaxe nos arquivos de configuração.

### Verificação de zonas

```bash
sudo named-checkzone
```

O comando pode ser utilizado para validar arquivos de zona específicos.

---

# 13. Consultas DNS

Para realizar consultas e verificar a resolução de nomes, podem ser utilizados:

```bash
dig
```

e:

```bash
nslookup
```

Exemplo:

```bash
dig cloudcraft.com.br
```

Para uma consulta reversa:

```bash
dig -x 192.168.12.2
```

Esses comandos permitem verificar se o servidor DNS está respondendo corretamente às consultas.

---

# 14. Logs

Os logs do BIND9 foram utilizados durante o laboratório para acompanhar o comportamento do serviço.

Entre os eventos observados estão:

* Inicialização do serviço;
* Escuta nas interfaces de rede;
* Notificações DNS;
* Recebimento de notificações;
* Atualização de zonas;
* Eventos relacionados à resolução externa.

Um dos registros observados na NS1 foi:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Enquanto a NS2 registrou:

```text
received notify for zone 'cloudcraft.com.br'
```

Esses registros foram importantes para validar a comunicação entre os servidores.

---

# 15. Conceitos praticados

A implementação permitiu praticar:

* DNS;
* BIND9;
* DNS primário;
* DNS secundário;
* Zona direta;
* Zona reversa;
* DNS NOTIFY;
* `in-addr.arpa`;
* Resolução de nomes;
* Consultas DNS;
* Logs de serviços;
* Administração de serviços Linux;
* Troubleshooting.

---

# 16. Resumo

A infraestrutura DNS implementada utiliza dois servidores BIND9 conectados através da rede interna `192.168.12.0/24`.

A NS1, localizada em `192.168.12.1`, atua como servidor DNS primário, enquanto a WEB/NS2, localizada em `192.168.12.2`, atua como servidor DNS secundário.

Durante os testes foram observadas notificações DNS entre os servidores para as zonas `cloudcraft.com.br` e `12.168.192.in-addr.arpa`, demonstrando a comunicação entre o servidor primário e o secundário.

O laboratório fornece uma base prática para o estudo de DNS, administração Linux e infraestrutura de servidores.
