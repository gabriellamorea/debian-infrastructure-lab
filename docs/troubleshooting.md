# 🛠️ Troubleshooting

## 1. Objetivo

Esta documentação registra problemas, comportamentos inesperados e mensagens identificadas durante a implementação e operação do laboratório.

O objetivo não é apenas registrar erros, mas demonstrar o processo de investigação utilizado para compreender o comportamento dos serviços.

---

# 2. Metodologia

O processo de troubleshooting utilizado no laboratório segue uma abordagem baseada em:

```text
Problema
   ↓
Coleta de informações
   ↓
Análise de logs
   ↓
Verificação da configuração
   ↓
Testes de conectividade
   ↓
Identificação da causa
   ↓
Correção ou validação
   ↓
Novo teste
```

Entre as principais ferramentas utilizadas estão:

* `ip addr`
* `ip route`
* `ping`
* `systemctl`
* `named-checkconf`
* `named-checkzone`
* `dig`
* `nslookup`
* Logs do BIND9

---

# 3. Mensagens relacionadas à resolução externa

Durante a execução do BIND9 na WEB/NS2 foram observadas mensagens como:

```text
broken trust chain resolving
```

Também foi registrado:

```text
resolver priming query complete: timed out
```

Essas mensagens estavam relacionadas a tentativas do BIND9 de realizar resolução DNS externa.

---

# 4. Análise do comportamento

Apesar das mensagens apresentadas anteriormente, o serviço BIND9 permaneceu ativo:

```text
named.service
Active: active (running)
```

Além disso, posteriormente foram observados eventos de comunicação bem-sucedida entre NS1 e NS2.

Isso indica que as mensagens registradas não impediram o funcionamento do serviço DNS interno utilizado pelo laboratório.

---

# 5. Comunicação DNS entre NS1 e NS2

Durante a análise dos logs, foi identificado na NS1:

```text
zone cloudcraft.com.br/IN:
sending notify to 192.168.12.2#53
```

Esse evento demonstrou que a NS1 estava enviando notificações DNS para a NS2.

Na NS2 foi identificado:

```text
received notify for zone 'cloudcraft.com.br'
```

E posteriormente:

```text
zone cloudcraft.com.br/IN:
notify from 192.168.12.1:
zone is up to date
```

---

# 6. Interpretação

Os registros acima demonstram um fluxo consistente:

```text
NS1
192.168.12.1
   │
   │ NOTIFY
   ▼
NS2
192.168.12.2
   │
   ▼
Verificação da zona
   │
   ▼
Zone is up to date
```

Portanto, a comunicação entre o DNS primário e o secundário foi observada diretamente através dos logs do BIND9.

---

# 7. Diagnóstico de rede

Em situações de falha de comunicação entre os servidores, o primeiro passo é verificar as interfaces:

```bash
ip addr
```

Depois, verificar as rotas:

```bash
ip route
```

A arquitetura esperada é:

```text
NS1
192.168.12.1
     │
     │
     ▼
Rede 192.168.12.0/24
     │
     │
     ▼
NS2
192.168.12.2
```

---

# 8. Teste de conectividade

Para verificar a comunicação básica entre os servidores:

Na NS1:

```bash
ping 192.168.12.2
```

Na NS2:

```bash
ping 192.168.12.1
```

Se esses testes falharem, devem ser investigados:

* Endereços IP;
* Máscaras;
* Interfaces;
* Estado das interfaces;
* Rede configurada no VirtualBox;
* Rotas;
* Firewall.

---

# 9. Diagnóstico do BIND9

Caso o DNS apresente problemas, verificar inicialmente o serviço:

```bash
sudo systemctl status bind9
```

Como o systemd apresenta o serviço como:

```text
named.service
```

também é possível utilizar:

```bash
sudo systemctl status named
```

---

# 10. Validação da configuração

Antes de reiniciar ou recarregar o BIND9, a configuração pode ser validada utilizando:

```bash
sudo named-checkconf
```

Esse procedimento ajuda a identificar erros de sintaxe nos arquivos de configuração.

---

# 11. Validação das zonas

Para verificar um arquivo de zona específico:

```bash
sudo named-checkzone <zona> <arquivo>
```

Por exemplo:

```bash
sudo named-checkzone cloudcraft.com.br <arquivo-da-zona>
```

O comando permite verificar a validade da estrutura da zona antes de colocá-la em produção.

---

# 12. Diagnóstico de resolução DNS

Para investigar problemas de resolução:

```bash
dig cloudcraft.com.br
```

Também pode ser utilizado:

```bash
nslookup cloudcraft.com.br
```

Para resolução reversa:

```bash
dig -x 192.168.12.2
```

Esses testes ajudam a identificar se o problema está relacionado à rede, ao serviço DNS ou à própria configuração da zona.

---

# 13. Análise dos logs

Os logs do BIND9 são uma das principais fontes de informação para investigação.

Alguns eventos relevantes observados no laboratório foram:

```text
sending notify
```

```text
received notify
```

```text
zone is up to date
```

Essas mensagens foram utilizadas para acompanhar a comunicação entre NS1 e NS2.

---

# 14. Boas práticas utilizadas

Durante a investigação, foram consideradas as seguintes práticas:

### 1. Verificar a rede antes do serviço

Primeiro validar:

```text
IP → rota → conectividade
```

Antes de investigar o DNS.

### 2. Verificar o estado do serviço

Utilizar:

```bash
systemctl status
```

para confirmar se o serviço está ativo.

### 3. Validar configuração antes de reiniciar

Utilizar:

```bash
named-checkconf
```

antes de realizar alterações no serviço.

### 4. Utilizar logs

Os logs permitem identificar o comportamento real do serviço em execução.

### 5. Repetir os testes

Após qualquer alteração:

```text
Alteração
   ↓
Teste
   ↓
Análise
```

---

# 15. Registro de incidentes

| Situação                          | Evidência          | Análise                                |
| --------------------------------- | ------------------ | -------------------------------------- |
| Mensagens de `broken trust chain` | Logs da NS2        | Relacionadas à resolução externa       |
| Timeout de resolver primário      | Logs da NS2        | Evento relacionado à resolução externa |
| NOTIFY enviado                    | Logs da NS1        | Comunicação com NS2 observada          |
| NOTIFY recebido                   | Logs da NS2        | Comunicação com NS1 confirmada         |
| Zona atualizada                   | Logs da NS2        | `zone is up to date`                   |
| BIND9 ativo                       | `systemctl status` | Serviço em execução                    |

---

# 16. Conclusão

O troubleshooting realizado durante o laboratório permitiu compreender melhor o comportamento do BIND9 e da comunicação entre servidores DNS.

As mensagens relacionadas à resolução externa foram analisadas em conjunto com o estado do serviço e com os eventos posteriores registrados nos logs.

A comunicação entre NS1 e NS2 foi confirmada através das mensagens de DNS NOTIFY e da indicação de que a zona `cloudcraft.com.br` estava atualizada.

O processo demonstrou a importância de utilizar informações de rede, estado dos serviços, validação de configuração e logs para investigar problemas em ambientes Linux.
