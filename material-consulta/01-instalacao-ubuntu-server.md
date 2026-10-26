# Roteiro 01: Introdução à Virtualização e Instalação do Ubuntu Server 22.04 LTS

## 1. Objetivos
* Compreender os conceitos básicos de virtualização (Hipervisor, Máquina Virtual, Isolamento de Recursos).
* Preparar o ambiente de trabalho local utilizando o Oracle VM VirtualBox.
* Realizar a instalação limpa e personalizada do sistema operacional de rede Ubuntu Server 22.04 LTS utilizando particionamento LVM.
* Validar a instalação por meio do primeiro login e atualização dos repositórios de pacotes.

---

## 2. Preparação do Ambiente de Trabalho (Host Windows)
Para garantir a organização e o desempenho dos computadores do laboratório, utilizaremos o drive `C:` local das máquinas host. Siga rigorosamente a estrutura de diretórios abaixo:

1. **Obter a Imagem ISO** (Ubuntu Server 22.04 LTS, mais leve para rodar as VMs com 512 MB de RAM):
   * **Opção A — download direto:** baixe a imagem em <https://releases.ubuntu.com/jammy/ubuntu-22.04.5-live-server-amd64.iso>.
   * **Opção B — servidor de arquivos do laboratório:**
     * Pressione as teclas `Win + R` no teclado do Windows.
     * Digite o caminho do servidor de arquivos do laboratório: `\\172.20.20.21\public` (usuário `alunoifal`, senha `alunoifal`) e pressione Enter.
     * Copie o arquivo de imagem `ubuntu-22.04.5-live-server-amd64.iso` para a sua máquina física.

2. **Organização de Pastas no Host:**
   * Crie o diretório de originais: `C:\2026.2\BSI\VM\original`.
   * Mova a imagem ISO baixada ou copiada para este diretório: `C:\2026.2\BSI\VM\original\ubuntu-22.04.5-live-server-amd64.iso`.
   * Crie o diretório de trabalho do aluno: `C:\2026.2\BSI\VM\<NomeDoAluno>` (substitua `<NomeDoAluno>` pelo seu primeiro e último nome, sem espaços ou acentos).
   * Dentro da sua pasta, crie a subpasta para a máquina virtual: `C:\2026.2\BSI\VM\<NomeDoAluno>\ubuntu_server`.

---

## 3. Criação e Configuração da VM no VirtualBox
Abra o Oracle VM VirtualBox e clique em **Novo** para configurar a máquina virtual com os seguintes parâmetros:

* **Nome:** `ubuntu_server`
* **Pasta da VM:** `C:\2026.2\BSI\VM\<NomeDoAluno>\ubuntu_server`
* **Tipo:** `Linux`
* **Versão:** `Ubuntu (64-bit)`
* **Tamanho de Memória (RAM):** `1536 MB` (1,5 GB) **durante a instalação**. Depois de instalar, a memória é reduzida para `512 MB` (seção 5).
* **Processador:** `1 CPU`
* **Disco Rígido:** Escolha "Criar um novo disco rígido virtual agora".
  * **Tipo de arquivo:** `VDI (VirtualBox Disk Image)`
  * **Armazenamento:** `Dinamicamente Alocado`
  * **Tamanho:** `32 GB`

---

## 4. Instalação Passo a Passo do Ubuntu Server
Com a VM criada, clique em **Configurações** -> **Armazenamento**. Em "Controladora: IDE", selecione o ícone de disco óptico vazio e monte o arquivo ISO localizado em `C:\2026.2\BSI\VM\original\ubuntu-22.04.5-live-server-amd64.iso`.

Inicie a máquina virtual e execute os seguintes passos no instalador interativo:

### 4.1. Idioma e Teclado
* **Language:** Selecione `English` (recomendado para servidores).
* **Keyboard Configuration:** Em Layout e Variant, configure para `Portuguese (Brazil)` ou o layout correspondente ao teclado físico do seu terminal.

### 4.2. Conexão de Rede
* O instalador identificará a interface `enp0s3` e tentará obter uma configuração de rede padrão automaticamente via DHCP.
* Verifique se ela exibe um endereço IP no formato `10.0.2.15/24` ou similar antes de avançar.

### 4.3. Proxy e Mirror
* **Proxy:** Deixe em branco (Apenas clique em `Done`).
* **Mirror:** Mantenha o endereço padrão do espelho do Ubuntu e clique em `Done`.

### 4.4. Particionamento do Sistema de Arquivos (Customizado LVM)
Não utilizaremos a instalação automática em disco inteiro. Selecione a opção **Custom storage layout** (ou manual) para criarmos um esquema de particionamento flexível utilizando LVM (Logical Volume Manager):

Criaremos as seguintes partições:
1. **Partição de Boot (/boot):**
   * **Tamanho:** `1 GB` (1024M)
   * **Formato:** `ext4`
   * **Ponto de Montagem:** `/boot`

2. **Grupo de Volumes LVM (ubuntu-vg):**
   * Crie o Volume Group com o restante do espaço em disco.

3. **Volume Lógico Root (/):**
   * **Nome:** `ubuntu-lv`
   * **Tamanho:** `29 GB`
   * **Formato:** `ext4`
   * **Ponto de Montagem:** `/`

4. **Volume Lógico Swap (SWAP):**
   * **Nome:** `swap-lv` (ou similar)
   * **Tamanho:** `2 GB` (ou o restante do espaço livre de ~2GB)
   * **Formato:** `swap`

*Selecione `Done` e confirme a gravação das alterações no disco.*

### 4.5. Configuração de Perfil (Profile Setup)
Insira as seguintes credenciais obrigatórias para padronização do laboratório:
* **Your name:** `Administrador`
* **Your server's name:** `ubuntu_server`
* **Pick a username:** `administrador`
* **Choose a password:** `adminifal`
* **Confirm your password:** `adminifal`

### 4.6. Ubuntu Pro
* Se o instalador exibir a tela **Upgrade to Ubuntu Pro**, escolha **Skip for now** e prossiga.

### 4.7. Serviços Adicionais (SSH)
* Na tela de SSH, marque a opção **[X] Install OpenSSH Server** pressionando a barra de espaço. Isso é crucial para as práticas de acesso remoto futuras.
* Na tela de snaps adicionais, não selecione nenhum serviço. Prossiga e aguarde a conclusão da instalação.
* Quando finalizar, selecione **Reboot Now** e pressione Enter quando o instalador solicitar a remoção da mídia de instalação.

---

## 5. Pós-Instalação e Validação
Após o reboot, faça login com o usuário criado:
```bash
ubuntu_server login: administrador
Password: <digite adminifal - não aparecerá na tela>
```

Execute a atualização inicial do catálogo de pacotes do sistema:
```bash
sudo apt-get update
```

### 5.1. Redução da memória da VM para 512 MB
O instalador do Ubuntu Server precisa de mais memória do que o sistema já instalado. Depois de concluir a instalação e a primeira atualização, reduza a memória da VM:

1. Desligue a VM:
   ```bash
   sudo poweroff
   ```
2. No VirtualBox, selecione a VM e abra **Configurações → Sistema → Placa-Mãe**.
3. Altere **Memória Base** de `1536 MB` para `512 MB` e clique em **OK**.
4. Ligue a VM novamente e confira a memória disponível:
   ```bash
   free -h
   ```

### 5.2. Extension Pack
No computador host (máquina física Windows), certifique-se de que o **VirtualBox Extension Pack** correspondente à versão instalada esteja devidamente instalado para garantir a compatibilidade de drivers e USB.

---

> **Vídeos de apoio** (operações com VMs no VirtualBox): [vídeo 1](https://youtu.be/z7BRXpNnCgA) · [vídeo 2](https://youtu.be/r0rZ58JTll0) · [vídeo 3](https://youtu.be/Js6yI43LvPU)

> **No projeto final:** cada integrante instala as VMs do seu PC seguindo este roteiro. Na tela *Profile Setup*, use em **Your server's name** o hostname curto da VM definido no planejamento do grupo (ex.: `g0-pc1-vm1`), e não deixe de marcar **Install OpenSSH Server**.
