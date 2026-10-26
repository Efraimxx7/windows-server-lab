# 02 — Arquitetura da Rede do Laboratório

## 1. Objetivo

Este documento apresenta a arquitetura lógica do laboratório Windows Server 2022, explicando os equipamentos, os endereços IP e o fluxo básico de comunicação entre o servidor e a estação cliente.

## 2. Diagrama lógico

```text
                   VIRTUALBOX
              Rede Host-only
               192.168.56.0/24
                       |
             +---------+---------+
             |                   |
             |                   |
      +-------------+      +-------------+
      | SRV-AD01    |      | WIN11       |
      | Windows     |      | Windows 11  |
      | Server 2022 |      | Pro         |
      +-------------+      +-------------+
      | IP .10      |      | IP .100     |
      | AD DS       |      | Cliente     |
      | DNS         |      | ingressado  |
      | DHCP        |      | no domínio  |
      | SMB         |      +-------------+
      +-------------+
```

## 3. Plano de endereçamento

| Elemento | Valor |
|---|---|
| Rede | 192.168.56.0/24 |
| Máscara | 255.255.255.0 |
| Servidor | 192.168.56.10 |
| Cliente (concessão observada) | 192.168.56.100 |
| Faixa DHCP | 192.168.56.100–192.168.56.150 |
| Domínio | corp.lab |
| DNS | 192.168.56.10 |

O servidor utiliza endereço estático. O cliente recebe um endereço do escopo DHCP, podendo receber outro endereço válido em uma concessão futura.

## 4. Função de cada serviço

### Active Directory Domain Services

Centraliza as contas de usuário e computador e permite autenticação e administração do domínio.

### DNS

Resolve nomes de computadores e serviços do domínio. O Active Directory depende de registros DNS adequados para que os clientes localizem o controlador de domínio.

### DHCP

Distribui automaticamente endereços IP e parâmetros de rede aos clientes, incluindo o servidor DNS e o sufixo DNS do domínio.

### Group Policy

Permite aplicar configurações administrativas e de segurança aos usuários e computadores que estão dentro do escopo de uma política vinculada.

### SMB e NTFS

SMB fornece o acesso aos compartilhamentos pela rede. NTFS controla as permissões sobre arquivos e pastas. As permissões efetivas dependem das regras das duas camadas.

## 5. Fluxo de comunicação

1. O cliente inicia a rede e solicita uma configuração IP ao DHCP.
2. O DHCP oferece uma concessão válida e informa o DNS do domínio.
3. O cliente consulta o DNS para localizar o servidor e os serviços do domínio.
4. O cliente utiliza o Active Directory para autenticação e localização do controlador de domínio.
5. As políticas de grupo aplicáveis são processadas pelo cliente.
6. Quando um usuário acessa um compartilhamento SMB, o servidor verifica a autenticação e as permissões de acesso.

## 6. Limites do laboratório

A rede Host-only permite testes de comunicação entre as máquinas virtuais e o hospedeiro. Ela não representa automaticamente uma rede corporativa com acesso à Internet ou a outras redes físicas.

O projeto foi criado para estudo e demonstração de conceitos de infraestrutura, e não como arquitetura pronta para produção.
