# Windows Server 2022 — Laboratório de Infraestrutura

## Sobre o projeto

Este projeto consiste em um laboratório prático de infraestrutura de TI desenvolvido em ambiente virtualizado com o objetivo de estudar e demonstrar a configuração e administração básica de serviços de rede em um ambiente Windows Server.

O laboratório simula uma pequena rede corporativa com um servidor central e uma estação cliente Windows 11 Pro integrada ao domínio.

O foco é demonstrar conhecimentos de administração de sistemas, gerenciamento centralizado de usuários, distribuição automática de endereços IP, resolução de nomes, aplicação de políticas de segurança e controle de acesso a arquivos.

## Objetivos

- Instalar e configurar o Windows Server 2022.
- Implementar um domínio com Active Directory Domain Services (AD DS).
- Organizar usuários, computadores e grupos por unidades organizacionais.
- Configurar o serviço DNS para resolução de nomes.
- Implementar DHCP para distribuição automática de endereços IP.
- Integrar uma estação Windows 11 Pro ao domínio.
- Criar e aplicar uma Group Policy Object (GPO).
- Configurar compartilhamentos SMB e permissões NTFS.
- Validar o acesso permitido e negado entre departamentos.
- Documentar a configuração e os testes realizados.

## Ambiente utilizado

| Componente | Configuração |
|---|---|
| Virtualização | Oracle VirtualBox |
| Servidor | Windows Server 2022 |
| Cliente | Windows 11 Pro |
| Domínio | `corp.lab` |
| Nome do servidor | `SRV-AD01` |
| Rede do laboratório | `192.168.56.0/24` |
| IP do servidor | `192.168.56.10` |
| Faixa DHCP | `192.168.56.100–192.168.56.150` |
| DNS | `192.168.56.10` |
| Tipo de rede | Host-only |

O ambiente utiliza uma rede virtual isolada do VirtualBox para permitir a comunicação entre o servidor e a estação cliente durante os testes.

## Arquitetura do laboratório

```text
              Rede virtual Host-only
                  192.168.56.0/24
                         |
             +-----------+-----------+
             |                       |
      +-------------+         +-------------+
      | SRV-AD01    |         | Windows 11  |
      | Windows     |         | Cliente     |
      | Server 2022 |         | DHCP        |
      +-------------+         +-------------+
      | IP: .10     |         | IP: .100    |
      | AD DS       |         | Ingressado  |
      | DNS         |         | no domínio  |
      | DHCP        |         | corp.lab    |
      | SMB         |         +-------------+
      +-------------+
```

## Serviços implementados

### Active Directory Domain Services (AD DS)

Foi criado o domínio `corp.lab` para centralizar o gerenciamento das contas de usuário e dos computadores do laboratório.

Estrutura organizacional:

```text
corp.lab
└── Empresa
    ├── Usuarios
    ├── Computadores
    └── Grupos
```

Foram criadas contas de usuário para simular funcionários de diferentes departamentos.

### Grupos de segurança

Foram criados grupos globais de segurança para organizar as permissões por departamento:

- `GG_TI`
- `GG_RH`
- `GG_FINANCEIRO`

A utilização de grupos facilita a administração das permissões sem precisar configurar cada usuário individualmente em todos os recursos.

### DNS

O serviço DNS foi configurado para permitir a resolução de nomes no domínio.

Testes realizados:

```cmd
nslookup srv-ad01.corp.lab
```

```cmd
nltest /dsgetdc:corp.lab
```

Os testes permitiram verificar a resolução do nome do servidor e a localização do controlador de domínio.

### DHCP

Foi criado e ativado o escopo `LAN-LAB`, responsável pela distribuição automática de endereços IPv4 para a estação cliente.

Configuração principal:

- Rede: `192.168.56.0/24`
- Faixa de endereços: `192.168.56.100–192.168.56.150`
- Servidor DHCP: `192.168.56.10`
- Servidor DNS distribuído: `192.168.56.10`
- Sufixo DNS: `corp.lab`

O cliente recebeu uma configuração de rede automaticamente por meio do DHCP.

### Group Policy Object (GPO)

Foi criada a política `GPO - Estacoes - Bloqueio de Tela` e vinculada à unidade organizacional destinada aos computadores do laboratório.

A configuração de segurança define um limite de inatividade da máquina de 600 segundos, equivalente a 10 minutos.

A aplicação da política foi verificada no cliente por meio do comando:

```cmd
gpresult /r /scope computer
```

O valor de configuração também foi consultado no Registro do Windows.

### Compartilhamento de arquivos e controle de acesso

Foram criados compartilhamentos SMB para simular os diretórios de três departamentos:

| Departamento | Pasta | Grupo de segurança |
|---|---|---|
| TI | `C:\Empresa\TI` | `GG_TI` |
| Recursos Humanos | `C:\Empresa\RH` | `GG_RH` |
| Financeiro | `C:\Empresa\Financeiro` | `GG_FINANCEIRO` |

As permissões NTFS foram configuradas para conceder acesso de modificação ao grupo correspondente, mantendo as permissões administrativas necessárias.

Os testes no Windows 11 verificaram que usuários conseguiam acessar as pastas autorizadas e recebiam acesso negado ao tentar abrir pastas de outros departamentos.

## Testes realizados

| Teste | Resultado |
|---|---|
| Resolução DNS do servidor | Validado |
| Localização do controlador de domínio | Validado |
| Distribuição automática de IP por DHCP | Validado |
| Ingresso do Windows 11 no domínio | Validado |
| Aplicação da GPO no cliente | Validado |
| Acesso autorizado à pasta do departamento | Validado |
| Bloqueio de acesso a outros departamentos | Validado |

## Evidências

As capturas de tela e os registros dos testes serão organizados no diretório `evidencias/`.

As evidências incluem configurações de rede, estrutura do Active Directory, grupos de segurança, testes de DNS e DHCP, aplicação da GPO, compartilhamentos e permissões NTFS.

## Competências praticadas

- Administração básica do Windows Server.
- Active Directory e gerenciamento de usuários.
- DNS e DHCP.
- Gerenciamento de políticas de grupo.
- Compartilhamento SMB.
- Permissões NTFS e controle de acesso.
- Diagnóstico e validação de serviços de rede.
- Virtualização com VirtualBox.
- Documentação técnica e organização de projetos.

## Limitações do ambiente

Este laboratório foi desenvolvido em uma rede virtual Host-only, destinada aos testes entre máquinas virtuais. Não representa uma infraestrutura corporativa em produção e não possui, por padrão, acesso externo fornecido por essa rede.

As configurações e os resultados apresentados referem-se ao ambiente de laboratório descrito neste repositório.

## 📸 Evidências do laboratório

As capturas de tela abaixo documentam as principais etapas da implementação e os testes realizados no ambiente Windows Server.

### Infraestrutura e rede
- [Configuração de rede do servidor](evidencias/01-servidor-configuracao-rede.png)
- [Configuração DHCP do cliente](evidencias/02-cliente-configuracao-dhcp.png)
- [Teste de resolução DNS](evidencias/10-teste-dns.png)
- [Descoberta do controlador de domínio](evidencias/11-teste-controlador-dominio.png)
- [Escopo DHCP configurado](evidencias/12-escopo-dhcp.png)

### Active Directory
- [Estrutura do domínio](evidencias/03-estrutura-dominio-ad.png)
- [Unidades organizacionais](evidencias/04-unidades-organizacionais.png)
- [Usuários do domínio](evidencias/05-usuarios-active-directory.png)
- [Grupos de segurança](evidencias/06-grupos-seguranca.png)

### Políticas de grupo (GPO)
- [Configuração da política de bloqueio de tela](evidencias/13-gpo-bloqueio-tela.png)
- [Confirmação da GPO aplicada ao cliente](evidencias/14-gpo-aplicada-cliente.png)

### Compartilhamentos e permissões
- [Pastas departamentais](evidencias/15-pastas-departamentos.png)
- [Compartilhamentos SMB](evidencias/16-compartilhamentos-smb.png)
- [Permissões NTFS — TI](evidencias/17-permissoes-ntfs-ti.png)
- [Permissões NTFS — RH](evidencias/18-permissoes-ntfs-rh.png)
- [Permissões NTFS — Financeiro](evidencias/19-permissoes-ntfs-financeiro.png)
- [Teste de acesso permitido ao compartilhamento TI](evidencias/20-acesso-ti-permitido.png)
- [Teste de acesso negado ao compartilhamento RH](evidencias/21-acesso-rh-negado-ti.png)
- [Teste de acesso negado ao compartilhamento Financeiro](evidencias/22-acesso-financeiro-negado-ti.png)


## Autor

Projeto desenvolvido para fins de estudo, prática técnica e construção de portfólio na área de Infraestrutura de TI.
