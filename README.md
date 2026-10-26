# Fundamentos de Redes de Computadores (FNRC) — BSI 2026.02

Repositório da disciplina **Fundamentos de Redes de Computadores** do curso de **Bacharelado em Sistemas de Informação (BSI)** no **Instituto Federal de Alagoas (IFAL) - Campus Maceió**, para o semestre letivo **2026.02**.

Aqui ficam as definições do **Projeto Final** e o **material de consulta** de apoio. A disciplina não tem aulas de Linux: os roteiros de consulta reúnem o passo a passo de cada habilidade prática usada no projeto, para que cada aluno possa revisar ou aprender no seu ritmo.

---

## 💻 Informações Gerais do Laboratório

*   **Hipervisor:** [VirtualBox](https://www.virtualbox.org/) (com o respetivo *Extension Pack* instalado no hospedeiro).
*   **Sistema Operacional Guest:** Ubuntu Server 26.04 LTS (64-bit).
*   **Hardware Padrão da VM:** 512 MB de Memória RAM, 1 CPU Virtual e 32 GB de Disco Rígido (VDI, Alocação Dinâmica).
*   **Usuário Administrativo Padrão:** `administrador`.
*   **Senha de Laboratório:** `adminifal`.

### 📁 Estrutura de Diretórios no Host (Windows do Laboratório)
*   **Diretório de ISOs originais:** `C:\2026\BSI\VM\original`.
*   **Diretório de Trabalho do Aluno:** `C:\2026\BSI\VM\<NomeDoAluno>`.
*   **Servidor de Arquivos da Rede (Acesso a ISO/VM):** `\\172.20.20.21\public` (Usuário: `alunoifal` | Senha: `alunoifal`)

---

## 🧩 Projeto Final

*   **[Projeto Final — Rede virtualizada com 8 VMs](projeto-final/README.md)**: Cada grupo monta uma rede com 8 VMs Ubuntu Server distribuídas em 4 PCs do laboratório, todas em modo Placa em Ponte. O Grupo X usa a sub-rede `192.168.26.(16 × X)/28`, com IP estático em todas as VMs, hostnames e FQDN no domínio `grupoX.bsi-26-2.maceio.lab`, `/etc/hosts` e os usuários dos integrantes em todas as máquinas. A entrega é feita no repositório do grupo, `fnrc-2026.2-bsi-pf-grupoX`, com as evidências dos testes de `ping`, `ssh` e do script de validação.

---

## 📚 Material de Consulta

Os roteiros abaixo trazem as explicações, os comandos e os testes de cada habilidade usada no projeto final.

1.  **[Roteiro 01: Instalação e Configuração Básica do Ubuntu Server](material-consulta/01-instalacao-ubuntu-server.md)**
    *   Criação de VM no VirtualBox.
    *   Particionamento avançado de disco usando LVM: `/` (29 GB), `/boot` (1 GB) e `SWAP` (2 GB).
    *   Instalação com o OpenSSH Server e primeira atualização de repositórios (`apt-get update`).
2.  **[Roteiro 02: Administração de Usuários, Grupos e Permissões](material-consulta/02-usuarios-grupos-permissoes.md)**
    *   Criação e gerenciamento de contas de usuários (`adduser`, `passwd`).
    *   Criação de grupos de trabalho e atribuição de membros.
    *   Permissões em diretórios compartilhados (`chown`, `chgrp`, `chmod`).
3.  **[Roteiro 03: Manipulação, Edição e Automação de Arquivos](material-consulta/03-arquivos-e-automacao-de-usuarios.md)**
    *   Edição de texto no terminal com os editores Nano e Vim.
    *   Comandos de manipulação (`touch`, `cp`, `mv`, `rm -i`) e leitura de arquivos (`cat`, `less`, `head`, `tail`).
    *   Scripts em Bash com laço `for` para criação de usuários em lote (`useradd -m`, `chpasswd`) e auditoria com `getent`.
4.  **[Roteiro 04: Acesso Remoto SSH e Diagnóstico de Rede](material-consulta/04-ssh-e-diagnostico-de-rede.md)**
    *   Modo NAT no VirtualBox e redirecionamento de portas (Host: `5222` -> Guest: `22`).
    *   Inspeção de rede com `netplan status`, `ifconfig`, `route -n`, `traceroute` e `w`.
    *   Monitoramento de portas no Windows Host com `netstat -an`.
5.  **[Roteiro 05: Placa em Ponte (Bridge) e IP Estático com Netplan](material-consulta/05-bridge-e-ip-estatico-netplan.md)**
    *   Mudança do adaptador de rede do VirtualBox de NAT para **Placa em Ponte (Bridge Adapter)**.
    *   IP estático no arquivo `/etc/netplan/00-installer-config.yaml` e aplicação com `sudo netplan apply`.
    *   Testes de conectividade com `ping` e `traceroute`.
6.  **[Roteiro 06: Nomes de Host, FQDN e /etc/hosts](material-consulta/06-nomes-de-host-e-etc-hosts.md)**
    *   Hostname curto, domínio e FQDN.
    *   Alteração do nome da máquina com `hostnamectl`.
    *   Resolução local de nomes com `/etc/hosts` e testes com `getent hosts`, `ping` e `ssh`.

---

## 🛠️ Dicas de Laboratório e Solução de Problemas

*   **Teclado Desconfigurado no Console:** Caso o layout do seu teclado esteja incorreto no terminal virtual da VM, configure-o para o padrão brasileiro ABNT2 com o comando:
    ```bash
    sudo dpkg-reconfigure keyboard-configuration
    ```
*   **VMs clonadas:** Ao clonar uma VM no VirtualBox, gere novos endereços MAC para as placas de rede. Duas VMs com o mesmo MAC no mesmo switch não se comunicam corretamente.
*   **Sem internet na rede do grupo:** A rede do projeto não tem gateway. Instale os pacotes necessários enquanto a placa da VM ainda está em modo NAT, antes de mudar para Placa em Ponte.

---

## 📖 Referências Bibliográficas Recomendadas

*   KUROSE, James F.; ROSS, Keith W. **Redes de Computadores e a Internet**. 8. ed. Pearson, 2022.
*   LACROIX, Jay. **Mastering Ubuntu Server**. 4. ed. Packt Publishing, 2023.
*   SOYINKA, Wale. **Linux Administration: A Beginner's Guide**. 8. ed. McGraw-Hill, 2020.
