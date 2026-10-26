# 01 — Implementação Técnica do Laboratório Windows Server

## 1. Visão geral

Este documento descreve a implementação de um ambiente virtualizado com Windows Server 2022 e Windows 11 Pro. O objetivo é praticar tarefas comuns de infraestrutura de TI, incluindo gerenciamento de domínio, serviços de rede, políticas de segurança e controle de acesso a arquivos.

## 2. Topologia e endereçamento IP

O laboratório utiliza uma rede virtual Host-only do Oracle VirtualBox. Essa modalidade permite que as máquinas virtuais e o computador hospedeiro se comuniquem pela rede virtual, sem depender de uma conexão direta com a rede física para os testes internos.

| Equipamento | Nome | Endereço IPv4 |
|---|---|---|
| Servidor | SRV-AD01 | 192.168.56.10 |
| Cliente | WIN11 | 192.168.56.100 |
| Rede | — | 192.168.56.0/24 |
| Máscara | — | 255.255.255.0 |

O servidor utiliza endereço IP estático, enquanto o cliente recebe seu endereço por DHCP.

O servidor não utiliza gateway padrão nessa interface isolada. O DNS do próprio controlador de domínio aponta para `127.0.0.1`.

## 3. Active Directory Domain Services

### 3.1 Domínio

Foi configurado o domínio:

`corp.lab`

O servidor `SRV-AD01` atua como controlador de domínio. O Active Directory centraliza as contas de usuários e computadores e permite aplicar políticas e administrar recursos de maneira centralizada.

### 3.2 Unidades organizacionais

A estrutura criada foi:

```text
corp.lab
└── Empresa
    ├── Usuarios
    ├── Computadores
    └── Grupos
```

As unidades organizacionais (OUs) ajudam a organizar os objetos do domínio e permitem direcionar políticas de grupo para conjuntos específicos de computadores ou usuários.

### 3.3 Contas de usuário

Foram criadas as seguintes contas para representar funcionários fictícios:

- `lucas.oliveira`
- `mariana.santos`
- `pedro.costa`

Essas contas foram utilizadas nos testes de autenticação e controle de acesso aos compartilhamentos departamentais.

### 3.4 Grupos de segurança

Foram criados três grupos globais de segurança:

| Grupo | Finalidade |
|---|---|
| GG_TI | Acesso aos recursos do departamento de TI |
| GG_RH | Acesso aos recursos de Recursos Humanos |
| GG_FINANCEIRO | Acesso aos recursos do Financeiro |

Atribuir permissões a grupos facilita a administração. Quando um funcionário muda de departamento, a associação aos grupos pode ser atualizada sem precisar redefinir individualmente todas as permissões dos recursos.

## 4. DNS

O Domain Name System (DNS) associa nomes a endereços IP e é fundamental para a localização de serviços do Active Directory.

O DNS do laboratório está no servidor `192.168.56.10`.

### Teste de resolução de nomes

No cliente Windows, foi executado:

```cmd
nslookup srv-ad01.corp.lab
```

O resultado esperado é a resolução de `srv-ad01.corp.lab` para `192.168.56.10`.

### Teste de localização do controlador de domínio

```cmd
nltest /dsgetdc:corp.lab
```

Esse comando solicita informações sobre um controlador de domínio para o domínio informado. O resultado permite verificar se o cliente consegue localizar o controlador de domínio.

## 5. DHCP

O Dynamic Host Configuration Protocol (DHCP) distribui automaticamente configurações de rede aos clientes.

Foi criado o escopo `LAN-LAB` com os seguintes parâmetros:

| Parâmetro | Valor |
|---|---|
| Faixa inicial | 192.168.56.100 |
| Faixa final | 192.168.56.150 |
| Máscara | 255.255.255.0 |
| DNS | 192.168.56.10 |
| Sufixo DNS | corp.lab |

O escopo foi ativado no servidor DHCP. O cliente recebeu um endereço IPv4 da faixa configurada, além das informações de DNS e do domínio.

### Verificação no cliente

```cmd
ipconfig /all
```

Esse comando permite consultar o endereço IPv4, a máscara, o servidor DHCP, o DNS e outras informações do adaptador de rede.

## 6. Group Policy Object (GPO)

Foi criada a política:

`GPO - Estacoes - Bloqueio de Tela`

A política foi vinculada à unidade organizacional destinada aos computadores do laboratório.

### Configuração

A configuração realizada foi:

`Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Interactive logon: Machine inactivity limit`

O valor configurado foi de 600 segundos, equivalente a 10 minutos.

### Verificação

No Windows 11, foi executado:

```cmd
gpresult /r /scope computer
```

Esse comando apresenta as políticas de computador aplicadas e permite verificar se a GPO esperada está sendo reconhecida pelo cliente.

Também foi consultado o valor de Registro:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v InactivityTimeoutSecs
```

O valor `0x258` equivale a 600 em decimal.

**Observação:** o valor do Registro confirma a configuração armazenada, mas um teste prático de inatividade e bloqueio é necessário para comprovar o comportamento final da estação.

## 7. Compartilhamentos SMB

Foram criadas três pastas departamentais:

| Caminho local | Nome do compartilhamento |
|---|---|
| `C:\Empresa\TI` | TI |
| `C:\Empresa\RH` | RH |
| `C:\Empresa\Financeiro` | Financeiro |

O protocolo SMB permite que os clientes acessem recursos compartilhados pela rede. No laboratório, os caminhos de acesso seguem o formato:

```text
\\SRV-AD01\TI
\\SRV-AD01\RH
\\SRV-AD01\Financeiro
```

## 8. Permissões NTFS e controle de acesso

As permissões NTFS controlam quais usuários e grupos podem ler, modificar ou administrar os arquivos e diretórios no sistema de arquivos.

As pastas foram configuradas com permissões de modificação para os grupos departamentais correspondentes:

| Pasta | Grupo autorizado |
|---|---|
| TI | GG_TI |
| RH | GG_RH |
| Financeiro | GG_FINANCEIRO |

As permissões de compartilhamento SMB e as permissões NTFS são camadas diferentes. Para acessar um recurso pela rede, as permissões efetivas precisam permitir a operação solicitada nas duas camadas.

As permissões administrativas e do sistema foram preservadas durante a configuração.

## 9. Testes de acesso

Foram utilizados os usuários do domínio para validar os acessos aos compartilhamentos.

| Usuário de teste | Recurso esperado | Resultado observado |
|---|---|---|
| Lucas Oliveira | TI | Acesso permitido |
| Lucas Oliveira | RH | Acesso negado |
| Lucas Oliveira | Financeiro | Acesso negado |
| Mariana Santos | RH | Acesso permitido |
| Mariana Santos | TI e Financeiro | Acesso negado |
| Pedro Costa | Financeiro | Acesso permitido |
| Pedro Costa | TI e RH | Acesso negado |

Esses testes demonstram a aplicação do princípio do menor privilégio: cada usuário deve possuir apenas as permissões necessárias para suas atividades.

## 10. Conclusão

O laboratório permitiu praticar a configuração de serviços centrais de uma infraestrutura Windows, incluindo Active Directory, DNS, DHCP, políticas de grupo e compartilhamentos SMB.

A integração da estação cliente ao domínio e os testes de acesso demonstraram como os serviços se complementam para fornecer autenticação centralizada, resolução de nomes, configuração de rede e controle de acesso.

Este projeto constitui um ambiente de estudo e demonstração de competências básicas de infraestrutura de TI, não uma implantação pronta para produção.

## 11. Evidências relacionadas

As capturas de tela correspondentes estão organizadas no diretório `evidencias/` do repositório. Elas documentam a configuração de rede, o domínio, os grupos de segurança, os testes de DNS, DHCP e GPO, os compartilhamentos e as permissões NTFS.
