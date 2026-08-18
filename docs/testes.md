# 🧪 Testes e Validação

## 1. Objetivo

Esta etapa tem como objetivo validar o funcionamento dos principais componentes do laboratório de infraestrutura.

Os testes foram direcionados para:

* Interfaces de rede;
* Tabelas de roteamento;
* Comunicação entre servidores;
* Estado do BIND9;
* Resolução DNS;
* Comunicação entre DNS primário e secundário;
* Zonas direta e reversa.

---

# 2. Validação das interfaces

O primeiro passo para validar a infraestrutura foi verificar as interfaces de rede de cada servidor.

O comando utilizado foi:

```bash
ip addr
```

Esse comando permite identificar:

* Interfaces disponíveis;
* Endereços IPv4;
* Endereços IPv6;
* Máscaras de rede;
* Estado das interfaces;
* Endereços MAC.

### NS1

Foram identificadas:

```text
enp0s3 → 10.0.2.15/24
enp0s8 → 192.168.12.1/24
```

### WEB / NS2

Foi identificada:

```text
enp0s3 → 192.168.12.2/24
```

A configuração confirma a separação entre a rede NAT e a rede interna.

---

# 3. Validação das rotas

Para verificar o roteamento de cada servidor foi utilizado:

```bash
ip route
```

## NS1

A NS1 apresentou:

```text
default via 10.0.2.2 dev enp0s3
10.0.2.0/24 dev enp0s3
192.168.12.0/24 dev enp0s8
```

Isso confirma:

* Gateway padrão `10.0.2.2`;
* Conectividade direta com `10.0.2.0/24`;
* Conectividade direta com `192.168.12.0/24`.

## WEB / NS2

A WEB apresentou:

```text
default via 192.168.12.1 dev enp0s3
192.168.12.0/24 dev enp0s3
```

Isso confirma que a NS1, através do endereço `192.168.12.1`, é utilizada como gateway da máquina.

---

# 4. Teste de conectividade

A comunicação entre os servidores pode ser validada utilizando:

```bash
ping
```

Na arquitetura do laboratório, a comunicação principal ocorre entre:

```text
NS1 → 192.168.12.1
NS2 → 192.168.12.2
```

O teste de conectividade entre os servidores tem como objetivo verificar se existe comunicação IP através da rede interna.

Exemplo:

```bash
ping 192.168.12.2
```

executado na NS1.

E:

```bash
ping 192.168.12.1
```

executado na NS2.

---

# 5. Validação do BIND9

O estado do serviço DNS foi verificado utilizando:

```bash
sudo systemctl status bind9
```

Embora o comando seja associado ao serviço BIND9, o systemd apresenta o serviço como:

```text
named.service
```

---

## 5.1 NS1

Na NS1 foi identificado:

```text
named.service
Active: active (running)
Loaded: loaded
enabled
```

Isso demonstra que o serviço está ativo e configurado para inicialização automática.

---

## 5.2 WEB / NS2

Na WEB também foi identificado:

```text
named.service
Active: active (running)
Loaded: loaded
enabled
```

Dessa forma, os dois servidores possuem o BIND9 em execução.

---

# 6. Validação da configuração

Para verificar a sintaxe da configuração do BIND9 pode ser utilizado:

```bash
sudo named-checkconf
```

Quando o comando não apresenta saída, isso normalmente indica que não foram encontrados erros de sintaxe na configuração analisada.

Esse teste é importante antes de reiniciar ou recarregar o serviço DNS.

---

# 7. Validação das zonas

As zonas podem ser verificadas através do comando:

```bash
sudo named-checkzone
```

Esse procedimento permite validar a estrutura dos arquivos de zona e identificar possíveis problemas de sintaxe ou configuração.

A zona principal utilizada no laboratório é:

```text
cloudcraft.com.br
```

Também foi identificada a zona reversa:

```text
12.168.192.in-addr.arpa
```

---

# 8. Teste de resolução DNS

Para realizar consultas DNS, podem ser utilizados:

```bash
dig
```

ou:

```bash
nslookup
```

Exemplo de consulta:

```bash
dig cloudcraft.com.br
```

Essas ferramentas permitem verificar:

* Qual servidor respondeu;
* Registros retornados;
* Tempo de resposta;
* Status da consulta;
* Informações da autoridade DNS.

---

# 9. Teste de resolução reversa

Para testar a resolução reversa de um endereço IP:

```bash
dig -x 192.168.12.2
```

Esse teste verifica se existe uma resposta associada ao endereço IP através da zona reversa:

```text
12.168.192.in-addr.arpa
```

---

# 10. Validação da comunicação DNS

Um dos principais testes realizados foi a análise dos logs do BIND9.

Na NS1 foi identificado:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Esse registro demonstra que o servidor primário enviou uma notificação para a NS2.

Na NS2 foi identificado:

```text
received notify for zone 'cloudcraft.com.br'
```

Posteriormente:

```text
zone cloudcraft.com.br/IN:
notify from 192.168.12.1:
zone is up to date
```

Esses registros fornecem evidências da comunicação entre os servidores DNS.

---

# 11. Validação da zona reversa

Também foi observada comunicação relacionada à zona:

```text
12.168.192.in-addr.arpa
```

A NS2 registrou o recebimento de uma notificação proveniente da NS1.

Esse comportamento demonstra que a comunicação entre os servidores também contempla a zona reversa.

---

# 12. Matriz de validação

| Teste              | Objetivo                  | Resultado observado                  |
| ------------------ | ------------------------- | ------------------------------------ |
| `ip addr`          | Verificar interfaces      | Interfaces configuradas              |
| `ip route`         | Verificar roteamento      | Rotas identificadas                  |
| `ping`             | Testar conectividade IP   | Comunicação prevista                 |
| `systemctl status` | Verificar BIND9           | Serviço ativo                        |
| `named-checkconf`  | Validar configuração      | Ferramenta disponível para validação |
| `named-checkzone`  | Validar zonas             | Ferramenta disponível para validação |
| `dig`              | Consultar DNS             | Utilizado para diagnóstico           |
| `nslookup`         | Consultar DNS             | Utilizado para diagnóstico           |
| Logs do BIND9      | Verificar comunicação DNS | NOTIFY observado                     |
| DNS reverso        | Validar zona reversa      | Zona identificada                    |

---

# 13. Evidências

As principais evidências técnicas obtidas durante o laboratório incluem:

### NS1

```text
10.0.2.15/24
192.168.12.1/24
```

### NS2

```text
192.168.12.2/24
```

### BIND9

```text
Active: active (running)
```

### DNS NOTIFY

```text
sending notify to 192.168.12.2#53
```

### Recebimento na NS2

```text
received notify for zone 'cloudcraft.com.br'
```

### Estado da zona

```text
zone is up to date
```

---

# 14. Resultado geral

Os testes realizados demonstraram que os principais componentes da infraestrutura estão configurados e operacionais.

A rede interna permite a comunicação entre os servidores, enquanto a NS1 possui conectividade adicional através da rede NAT.

O BIND9 encontra-se ativo nos dois servidores e os logs demonstraram comunicação entre o DNS primário e o secundário através do mecanismo DNS NOTIFY.

A infraestrutura também possui uma zona direta (`cloudcraft.com.br`) e uma zona reversa (`12.168.192.in-addr.arpa`).

---

# 15. Próximas validações

Como evolução do laboratório, novos testes podem ser adicionados para aprofundar a validação:

* [ ] Testar consultas DNS diretamente na NS1;
* [ ] Testar consultas DNS diretamente na NS2;
* [ ] Validar registros `A`;
* [ ] Validar registros `NS`;
* [ ] Validar registros `SOA`;
* [ ] Validar registros `PTR`;
* [ ] Testar atualização da zona;
* [ ] Confirmar atualização da zona secundária após alteração na primária;
* [ ] Testar comportamento após reinicialização dos servidores;
* [ ] Testar indisponibilidade temporária da NS1;
* [ ] Documentar tempos de resposta DNS.
