# Roteiro 06: Nomes de Host, FQDN e Resolução Local com /etc/hosts

Neste roteiro, você verá como dar um nome a cada máquina virtual com o `hostnamectl` e como fazer as máquinas de uma mesma rede se encontrarem pelo nome, sem um servidor DNS, usando o arquivo `/etc/hosts`.

---

## 1. Objetivos
* Diferenciar hostname curto, domínio e FQDN (*Fully Qualified Domain Name*).
* Alterar o nome de uma máquina Ubuntu Server com o `hostnamectl`.
* Mapear nomes para endereços IP no arquivo `/etc/hosts`.
* Verificar a resolução de nomes com `getent hosts` e `ping`.

---

## 2. Fundamentos: Hostname, Domínio e FQDN

*   **Hostname curto:** o nome da máquina, sem domínio. Exemplo: `g0-pc1-vm1`.
*   **Domínio:** o sufixo que identifica a rede ou organização a que a máquina pertence. Exemplo: `grupo0.bsi-26-2.maceio.lab`.
*   **FQDN:** o nome completo, formado pelo hostname curto seguido do domínio. Exemplo: `g0-pc1-vm1.grupo0.bsi-26-2.maceio.lab`.

Regras práticas para nomes de host:
*   Use apenas letras minúsculas, números e hífen (`-`).
*   Não use espaços, acentos nem sublinhado (`_`).
*   Não comece nem termine o nome com hífen.

Em redes grandes, a tradução de nomes em endereços IP é feita por um servidor DNS. Em uma rede pequena e sem DNS, como a rede de um grupo no laboratório, cada máquina pode guardar essa tradução localmente no arquivo `/etc/hosts`. O sistema consulta esse arquivo antes de recorrer ao DNS.

---

## 3. Alterando o Nome da Máquina com `hostnamectl`

Verifique o nome atual:
```bash
administrador@ubuntu_server:~$ hostnamectl
administrador@ubuntu_server:~$ hostname
```

Altere o nome para o FQDN da máquina:
```bash
administrador@ubuntu_server:~$ sudo hostnamectl set-hostname g0-pc1-vm1.grupo0.bsi-26-2.maceio.lab
```

Confira o resultado:
```bash
administrador@ubuntu_server:~$ hostname
g0-pc1-vm1.grupo0.bsi-26-2.maceio.lab
administrador@ubuntu_server:~$ hostname -s
g0-pc1-vm1
administrador@ubuntu_server:~$ cat /etc/hostname
g0-pc1-vm1.grupo0.bsi-26-2.maceio.lab
```

O prompt do terminal só mostra o novo nome depois de sair e entrar novamente (`exit` e novo login).

### 3.1 Nome da máquina × identificador da máquina (`machine-id`)

O **hostname** é o nome da máquina **para as pessoas**: aparece no prompt, nos logs e é usado para encontrar a máquina na rede. Além dele, cada instalação do Linux tem um **identificador interno**, o **`machine-id`**: um número hexadecimal de 32 caracteres, gravado no arquivo `/etc/machine-id` e criado automaticamente no primeiro boot do sistema. Ele também aparece na saída do `hostnamectl`, na linha `Machine ID`.

Os dois são **independentes**: trocar o hostname **não** muda o `machine-id`.

**Por que isso importa?** Quando uma VM é **clonada** no VirtualBox, ou importada de um mesmo arquivo `.ova`, a cópia herda o `machine-id` da original, mesmo recebendo um nome e um endereço MAC novos. O Ubuntu Server usa esse identificador, por exemplo, para montar o **identificador de cliente DHCP**: duas VMs clonadas, na mesma rede e usando DHCP, podem receber **o mesmo endereço IP**, porque o servidor DHCP enxerga as duas como a mesma máquina.

Veja o `machine-id` em cada VM:

```bash
administrador@g0-pc1-vm1:~$ cat /etc/machine-id
```

Se duas VMs do grupo tiverem valores **iguais**, uma delas foi clonada da outra. Gere um identificador novo **somente na VM clonada** (no exemplo, a `g0-pc1-vm2`):

**Passo 1.** Apague o identificador herdado e também a cópia dele usada pelo D-Bus (`/var/lib/dbus/machine-id`). Se essa cópia for um arquivo comum, e não um link, o `systemd-machine-id-setup` do passo 2 copia o ID antigo de volta a partir dela:

```bash
administrador@g0-pc1-vm2:~$ sudo rm -f /etc/machine-id /var/lib/dbus/machine-id
```

**Passo 2.** Gere um identificador novo e único. O `systemd-machine-id-setup` é a ferramenta do `systemd` que cria o `/etc/machine-id` quando ele não existe:

```bash
administrador@g0-pc1-vm2:~$ sudo systemd-machine-id-setup
```

**Passo 3.** Recrie a cópia do D-Bus como um link para o arquivo novo, para os dois ficarem sempre iguais:

```bash
administrador@g0-pc1-vm2:~$ sudo ln -sf /etc/machine-id /var/lib/dbus/machine-id
```
**Passo 4.** Reinicie a VM para que todos os serviços passem a usar o novo identificador:

```bash
administrador@g0-pc1-vm2:~$ sudo reboot
```

Depois do boot, rode `cat /etc/machine-id` de novo e confira que agora os valores são diferentes.

> Se as VMs foram instaladas do zero, uma a uma, os identificadores já são diferentes e não é preciso fazer nada.

---

## 4. Mapeando Nomes no Arquivo `/etc/hosts`

Abra o arquivo com privilégios de administrador:
```bash
administrador@g0-pc1-vm1:~$ sudo nano /etc/hosts
```

Cada linha do arquivo segue o formato:
```text
ENDEREÇO_IP    FQDN    apelido1 [apelido2 ...]
```

### Passo 4.1: Ajustar a linha da própria máquina
A instalação cria uma linha `127.0.1.1` com o nome antigo da máquina. Altere-a para o novo nome:
```text
127.0.1.1    g0-pc1-vm1.grupo0.bsi-26-2.maceio.lab    g0-pc1-vm1
```

### Passo 4.2: Acrescentar as demais máquinas da rede
No final do arquivo, acrescente uma linha para cada máquina da rede. Exemplo com quatro máquinas:
```text
# Rede do grupo 0 - grupo0.bsi-26-2.maceio.lab
192.168.26.1    g0-pc1-vm1.grupo0.bsi-26-2.maceio.lab    g0-pc1-vm1
192.168.26.2    g0-pc1-vm2.grupo0.bsi-26-2.maceio.lab    g0-pc1-vm2
192.168.26.3    g0-pc2-vm1.grupo0.bsi-26-2.maceio.lab    g0-pc2-vm1
192.168.26.4    g0-pc2-vm2.grupo0.bsi-26-2.maceio.lab    g0-pc2-vm2
```

Salve com `Ctrl+O`, confirme com `Enter` e saia com `Ctrl+X`.

> **Atenção:** o arquivo `/etc/hosts` vale apenas para a máquina onde foi editado. Para que todas as máquinas se encontrem pelo nome, o mesmo bloco de linhas precisa ser acrescentado em **cada uma** delas.

---

## 5. Testes de Resolução de Nomes

Consulte a tradução de um nome sem enviar pacotes pela rede:
```bash
administrador@g0-pc1-vm1:~$ getent hosts g0-pc2-vm1
192.168.26.3    g0-pc2-vm1.grupo0.bsi-26-2.maceio.lab g0-pc2-vm1
```

Teste a conectividade usando o apelido e o FQDN:
```bash
administrador@g0-pc1-vm1:~$ ping -c 3 g0-pc2-vm1
administrador@g0-pc1-vm1:~$ ping -c 3 g0-pc2-vm1.grupo0.bsi-26-2.maceio.lab
```

Acesse outra máquina por SSH usando o nome:
```bash
administrador@g0-pc1-vm1:~$ ssh administrador@g0-pc2-vm1
```

Se `getent hosts` não retornar nada, revise a grafia do nome no `/etc/hosts`. Se o nome for resolvido mas o `ping` falhar, o problema está na configuração de rede (endereço IP, máscara ou modo da placa no VirtualBox), e não no nome.

---

## 6. Guia de Comandos Utilizados

*   **`hostnamectl`:** exibe e altera o nome da máquina de forma permanente.
*   **`hostname` / `hostname -s`:** mostram o nome atual completo e o nome curto.
*   **`/etc/hosts`:** arquivo de mapeamento estático entre endereços IP e nomes.
*   **`getent hosts <nome>`:** consulta a tradução de um nome usando a mesma ordem de busca do sistema (`/etc/hosts` e depois DNS).
*   **`/etc/machine-id`:** arquivo com o identificador interno e único da instalação, independente do hostname.
*   **`systemd-machine-id-setup`:** gera um `machine-id` novo quando o arquivo não existe (usado em VMs clonadas). Se existir o `/var/lib/dbus/machine-id`, ele copia o ID desse arquivo; por isso, numa VM clonada, os dois são apagados antes.

---

> **No projeto final:** cada VM recebe o FQDN definido no planejamento do grupo, e o `/etc/hosts` de **todas as 8 VMs** contém as 8 máquinas do grupo (veja o [README do projeto](../projeto-final/README.md)).

---

## Referências

* FREEDESKTOP.ORG. **machine-id(5)**: identificador local da máquina. Disponível em: <https://www.freedesktop.org/software/systemd/man/latest/machine-id.html>.
* FREEDESKTOP.ORG. **systemd-machine-id-setup(1)**. Disponível em: <https://www.freedesktop.org/software/systemd/man/latest/systemd-machine-id-setup.html>.
* FREEDESKTOP.ORG. **systemd.network(5)**, opção `ClientIdentifier=` da seção `[DHCPv4]`. Disponível em: <https://www.freedesktop.org/software/systemd/man/latest/systemd.network.html>.
