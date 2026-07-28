# Instalação Automatizada do Windows

<img src="https://img.shields.io/badge/Windows-10-0078D4?logo=windows&logoColor=white" alt="Windows 10"> <img src="https://img.shields.io/badge/Windows-11-0078D4?logo=windows&logoColor=white" alt="Windows 11"> <img src="https://img.shields.io/badge/Firmware-UEFI-107C10" alt="UEFI"> <img src="https://img.shields.io/badge/Partition-GPT-107C10" alt="GPT"> <img src="https://img.shields.io/badge/License-MIT-yellow" alt="License">

Instale o Windows 10 ou 11 **sem clicar em nada**: você inicia o computador pela mídia de instalação e, de 10 a 30 minutos depois, ele entrega a Área de Trabalho pronta — em português, com teclado ABNT2, conta local criada e ajustes de privacidade aplicados.

Tudo isso é feito por um único arquivo de configuração, o `Autounattend.xml`, colocado na raiz do pendrive ou da ISO. Não é preciso instalar nenhum programa nem saber programar.

> **Novo por aqui?** Siga esta ordem: [Avisos importantes](#1-avisos-importantes) → [Pré-requisitos](#2-pré-requisitos) → [Guia de instalação](#3-guia-de-instalação) → [Verificação final](#6-verificação-pós-instalação). Se algo der errado, vá direto à [Solução de problemas](#7-solução-de-problemas).

---

## Sumário

1. [Avisos importantes](#1-avisos-importantes)
2. [Pré-requisitos](#2-pré-requisitos)
3. [Guia de instalação](#3-guia-de-instalação)
   * [3.1 Preparar a mídia (pendrive)](#31-preparar-a-mídia--pendrive-computador-físico)
   * [3.2 Preparar a mídia (ISO para máquina virtual)](#32-preparar-a-mídia--iso-máquina-virtual)
   * [3.3 Executar a instalação](#33-executar-a-instalação)
4. [Instalação em máquinas virtuais](#4-instalação-em-máquinas-virtuais)
5. [Personalizações comuns](#5-personalizações-comuns)
6. [Verificação pós-instalação](#6-verificação-pós-instalação)
7. [Solução de problemas](#7-solução-de-problemas)
8. [Como funciona (detalhes técnicos)](#8-como-funciona-detalhes-técnicos)
9. [Compatibilidade](#9-compatibilidade)
10. [Estrutura do projeto e licença](#10-estrutura-do-projeto-e-licença)

---

## 1. Avisos importantes

Leia os quatro avisos abaixo antes de qualquer outra coisa. Eles evitam perda de dados e frustrações.

### 1.1 O disco será apagado por completo

> [!WARNING]
> Este arquivo apaga **totalmente o Disco 0** (normalmente o disco principal) **sem pedir confirmação**. Todos os arquivos, fotos e programas serão perdidos.
>
> Use somente com **backup feito** ou em computadores **sem dados importantes**. Se houver mais de um disco no computador, veja o [Problema 7](#problema-7--disco-errado-foi-formatado-ou-medo-de-formatar-o-errado) **antes** de instalar.

### 1.2 O computador precisa estar em modo UEFI

> [!IMPORTANT]
> A configuração padrão exige firmware **UEFI** (padrão em praticamente todos os computadores fabricados a partir de ~2012 e exigido pelo Windows 11).
>
> Computadores muito antigos em modo **Legacy BIOS/CSM** precisam do bloco alternativo que está comentado no final do próprio `Autounattend.xml`, com instruções de uso.

### 1.3 A conta criada não tem senha

> [!CAUTION]
> Por conveniência, a conta `UserLocal` é criada **sem senha** e com **login automático** — ótimo para testes e máquinas virtuais, **inseguro para uso diário**.
>
> Após instalar, crie uma senha em `Configurações → Contas → Opções de entrada → Senha`.

### 1.4 Sobre o bypass de requisitos do Windows 11

> [!NOTE]
> O arquivo dispensa as verificações de TPM 2.0, Secure Boot, CPU, RAM e armazenamento, permitindo instalar o Windows 11 em máquinas fora da lista oficial. Funciona na grande maioria dos casos, mas máquinas fora dos requisitos podem não receber todas as atualizações da Microsoft.
>
> **Limite físico:** a partir do Windows 11 **24H2**, processadores anteriores a ~2009 (sem instruções SSE4.2) não funcionam de forma alguma — nenhum bypass resolve.

---

## 2. Pré-requisitos

### O que você vai precisar

| Item | Detalhes |
| --- | --- |
| ISO oficial do Windows | Baixe no site da Microsoft ("Baixar imagem de disco do Windows 10/11"). As ISOs oficiais contêm a edição Pro, usada por padrão neste projeto. |
| Pendrive de 8 GB ou mais | Apenas para instalação em computador físico. Será apagado. |
| Rufus (gratuito) | Apenas para gravar o pendrive. Alternativa: Media Creation Tool. |
| AnyBurn ou UltraISO | Apenas para instalação em máquina virtual (editar a ISO). |
| O arquivo `Autounattend.xml` | Fornecido neste projeto. |

> Desconecte o cabo de rede durante a instalação do Windows para evitar a execução do OOBE com integração à Microsoft Cloud

### Lista de verificação (evita 90% dos problemas)

Antes de iniciar a instalação, confirme cada item:

- [ ] Fiz **backup** de tudo que importa — o disco será apagado sem confirmação;
- [ ] O arquivo se chama exatamente `Autounattend.xml` (não `Autounattend.xml.txt` — ative `Exibir → Extensões de nomes de arquivo` no Explorador para conferir);
- [ ] O arquivo está na **raiz** da mídia, fora de qualquer pasta;
- [ ] O computador ou a VM está em modo **UEFI**;
- [ ] Somente o disco de destino está conectado (ou tenho certeza de que ele é o Disco 0);
- [ ] A ISO é oficial da Microsoft;
- [ ] No Rufus, **desmarquei** todas as personalizações automáticas (detalhes na próxima seção);
- [ ] De preferência, vou **testar primeiro em uma máquina virtual** ([seção 4](#4-instalação-em-máquinas-virtuais)).

---

## 3. Guia de instalação

Escolha o caminho conforme o destino: **pendrive** para computador físico (3.1) ou **ISO editada** para máquina virtual (3.2). Depois, siga para a execução (3.3).

### 3.1 Preparar a mídia — pendrive (computador físico)

1. Baixe a ISO oficial do Windows no site da Microsoft.
2. Abra o **Rufus**, selecione o pendrive e a ISO, e configure:
   * **Esquema de partição:** `GPT`
   * **Sistema de destino:** `UEFI (não CSM)`
3. Clique em Iniciar. Quando aparecer a janela **"Windows User Experience"** (perguntando sobre remover requisitos, criar conta local etc.), **desmarque todas as opções** — se ficarem marcadas, o Rufus cria um arquivo de resposta próprio que conflita com o deste projeto.
4. Ao terminar a gravação, copie o `Autounattend.xml` para a **raiz do pendrive**, ao lado das pastas do Windows:

   ```
   Pendrive
   │
   ├── Autounattend.xml   ← aqui, no primeiro nível
   ├── boot
   ├── efi
   ├── sources
   └── setup.exe
   ```

### 3.2 Preparar a mídia — ISO (máquina virtual)

Máquinas virtuais não usam pendrive: o arquivo precisa estar dentro da ISO (ou em uma segunda ISO — veja a dica abaixo).

1. Abra a ISO oficial do Windows no **AnyBurn** (gratuito) ou **UltraISO**.
2. Adicione o `Autounattend.xml` na **raiz** da imagem.
3. Salve como uma nova ISO. Esses programas preservam automaticamente as informações de boot ao editar uma ISO existente.

> [!TIP]
> **Atalho sem editar a ISO:** o instalador procura o `Autounattend.xml` na raiz de **todas as unidades conectadas**. Você pode criar uma segunda ISO minúscula contendo apenas o arquivo e conectá-la como segundo drive de CD/DVD da VM. Funciona no VirtualBox, VMware e Hyper-V.

### 3.3 Executar a instalação

1. **Computador físico:** conecte o pendrive, ligue o computador e pressione a tecla do menu de boot (`F12`, `F11`, `F9` ou `ESC`, conforme o fabricante). Escolha a entrada do pendrive **com o prefixo "UEFI"** (ex.: `UEFI: SanDisk`).
   **Máquina virtual:** conecte a ISO e inicie a VM (configuração da VM na [seção 4](#4-instalação-em-máquinas-virtuais)).
2. A instalação começa sozinha. O que esperar:

   | Etapa | O que aparece na tela | Duração típica |
   | --- | --- | --- |
   | Cópia dos arquivos | Tela azul com barra de progresso | 5–15 min |
   | 1ª reinicialização | Tela preta e logotipo do Windows | — |
   | Preparação | "Preparando…" / "Aguarde" com mais 1–2 reinicializações | 5–15 min |
   | Conclusão | Entra sozinho na Área de Trabalho, logado como `UserLocal` | — |

3. **Único momento que exige atenção:** na **primeira reinicialização**, remova o pendrive (ou desconecte a ISO da VM) para o computador não recomeçar a instalação.
4. Não toque no computador no restante do processo. Ao final, siga a [verificação pós-instalação](#6-verificação-pós-instalação).

---

## 4. Instalação em máquinas virtuais

Testar em uma máquina virtual antes de usar em um computador real é **gratuito, seguro e recomendado para todos**. Configuração correta por programa:

### VirtualBox (gratuito)

| Passo | Configuração |
| --- | --- |
| 1 | Crie uma VM do tipo "Windows 11 (64-bit)". |
| 2 | Marque **"Skip Unattended Installation"** — o instalador automático do próprio VirtualBox conflita com este projeto. |
| 3 | Em `Configurações → Sistema → Placa-mãe`, confirme **"Habilitar EFI"** marcado (no perfil Windows 10, marque manualmente). |
| 4 | Reserve **4 GB de RAM** e **64 GB de disco**. |
| 5 | Conecte a ISO preparada na seção 3.2 e inicie. |

### VMware Workstation / Player

| Passo | Configuração |
| --- | --- |
| 1 | Ao criar a VM, escolha **"I will install the operating system later"** — isso evita o "Easy Install" da VMware, que conflita com este projeto. |
| 2 | Selecione o perfil Windows 10/11 x64 (firmware UEFI já é o padrão). |
| 3 | Depois de criada, conecte a ISO preparada no drive de CD/DVD e inicie. |

### Hyper-V (incluído no Windows Pro)

| Passo | Configuração |
| --- | --- |
| 1 | Crie a VM como **Geração 2** (a Geração 1 usa BIOS Legacy e não funciona). |
| 2 | Se usar Memória Dinâmica, defina **RAM inicial de 4096 MB** — com pouca RAM o instalador falha antes do bypass agir. |
| 3 | Conecte a ISO preparada e inicie. Secure Boot pode ficar ligado ou desligado. |

### QEMU / KVM / Proxmox

| Passo | Configuração |
| --- | --- |
| 1 | Use firmware **OVMF (UEFI)**, não SeaBIOS. |
| 2 | Configure o disco virtual como **SATA**. Discos **VirtIO** não são detectados pelo instalador sem drivers ([Problema 5](#problema-5--o-disco-não-aparece-para-o-instalador)). |
| 3 | Conecte a ISO preparada e inicie. |

---

## 5. Personalizações comuns

O arquivo funciona sem nenhuma alteração. Se quiser personalizar, edite **apenas com o Bloco de Notas ou Notepad++** (nunca Word/WordPad), altere somente o texto **entre** as tags e valide depois arrastando o arquivo para o navegador — se abrir sem erro, está íntegro.

| Quero mudar… | Onde alterar no XML | Valor |
| --- | --- | --- |
| Edição para **Home** | Texto entre `<Key>` e `</Key>` | `YTMG3-N6DKC-DKB77-7M9GH-8HVX7` |
| Teclado para **US Internacional** | As **duas** linhas `<InputLocale>` | `0416:00020409` |
| Teclado para **ABNT (antigo)** | As **duas** linhas `<InputLocale>` | `0416:00000416` |
| Nome do usuário | `<Name>`, `<DisplayName>`, `<Username>` (e opcionalmente `<FullName>` e `<RegisteredOwner>`) | Mesmo nome em **todas** as ocorrências |
| Disco de destino | As **duas** ocorrências de `<DiskID>` | Número visto no `diskpart` ([Problema 7](#problema-7--disco-errado-foi-formatado-ou-medo-de-formatar-o-errado)) |
| Fuso horário | As **duas** linhas `<TimeZone>` | Nome exato obtido com o comando `tzutil /l` |
| Computador antigo **BIOS Legacy/MBR** | Bloco alternativo comentado no final do XML | Instruções no próprio bloco |

Sobre a chave padrão (`W269N-WFGWX-YVC9B-4J6C9-T83GX`): é uma **chave genérica pública da Microsoft (GVLK)**, usada apenas para o instalador selecionar a edição **Pro** automaticamente. Ela **não ativa o Windows** e não é pirataria — a ativação continua sendo feita depois, com a sua licença.

---

## 6. Verificação pós-instalação

Ao chegar à Área de Trabalho, confirme que tudo saiu como esperado:

| # | Verificação | Como fazer | Resultado esperado |
| --- | --- | --- | --- |
| 1 | Conta criada | Menu Iniciar → ícone do usuário | Conta `UserLocal` (Administrador) |
| 2 | Login automático | Reinicie o computador | Entra na Área de Trabalho sem pedir senha |
| 3 | Idioma e teclado | Digite `ç` e acentos em um bloco de notas | Caracteres corretos (ABNT2) |
| 4 | Modo UEFI | `Win + R` → `msinfo32` → "Modo da BIOS" | `UEFI` |
| 5 | Disco e partições | Clique direito no Iniciar → `Gerenciamento de Disco` | Partições System/MSR/Windows (C:) no disco certo |
| 6 | Drivers | Clique direito no Iniciar → `Gerenciador de Dispositivos` | Sem alertas amarelos ([Problema 15](#problema-15--sem-internet-ou-drivers-faltando) se houver) |
| 7 | Fuso horário | Relógio da barra de tarefas | Horário de Brasília |

**Recomendações finais:** crie uma senha para a conta (`Configurações → Contas → Opções de entrada`), conecte à internet e rode o Windows Update, e ative o Windows com a sua licença.

---

## 7. Solução de problemas

### Encontre seu problema pelo sintoma

| O que está acontecendo | Vá para |
| --- | --- |
| A instalação pede idioma/teclado (não está automática) | [Problema 1](#problema-1--a-instalação-não-está-automática) |
| Erro de chave de produto | [Problema 2](#problema-2--erro-de-chave-de-produto) |
| Erro citando "arquivo de resposta autônoma" | [Problema 3](#problema-3--erro-o-windows-não-pode-processar-o-arquivo-de-resposta) |
| A instalação recomeça do zero em loop | [Problema 4](#problema-4--a-instalação-recomeça-em-loop-após-reiniciar) |
| Nenhum disco aparece / erro ao criar partição | [Problema 5](#problema-5--o-disco-não-aparece-para-o-instalador) |
| Tela "Este computador não pode executar o Windows 11" | [Problema 6](#problema-6--a-tela-de-requisitos-do-windows-11-aparece-mesmo-assim) |
| Disco errado formatado / vários discos no computador | [Problema 7](#problema-7--disco-errado-foi-formatado-ou-medo-de-formatar-o-errado) |
| "No bootable device" após instalar | [Problema 8](#problema-8--o-computador-não-inicia-após-a-instalação) |
| Problemas no VirtualBox | [Problema 9](#problema-9--virtualbox-não-automatiza-ou-não-dá-boot) |
| Problemas no VMware (Easy Install) | [Problema 10](#problema-10--vmware-easy-install-quebra-a-automação) |
| Problemas no Hyper-V | [Problema 11](#problema-11--hyper-v-falha-no-início-ou-não-inicializa) |
| Pede conta Microsoft/internet no final | [Problema 12](#problema-12--o-windows-pede-conta-microsoft-mesmo-assim) |
| Para na tela de login (autologon não funciona) | [Problema 13](#problema-13--o-login-automático-não-funciona) |
| Teclado digitando símbolos errados | [Problema 14](#problema-14--teclado-digitando-símbolos-errados) |
| Sem internet / drivers faltando | [Problema 15](#problema-15--sem-internet-ou-drivers-faltando) |
| Erro no passe "specialize" | [Problema 16](#problema-16--erro-durante-o-passe-specialize) |
| Tela preta / travamento no primeiro boot | [Problema 17](#problema-17--tela-preta-ou-travamento-no-primeiro-boot) |
| Erro de espaço em disco | [Problema 18](#problema-18--disco-pequeno-demais) |
| Instalação saiu diferente do descrito aqui | [Problema 19](#problema-19--o-rufus-criou-configurações-próprias) |
| Outro erro qualquer | [Problema 20](#problema-20--investigando-qualquer-outro-erro-logs) |

Cada problema segue a mesma estrutura: **o que acontece → causa → como confirmar → solução → como evitar → como validar**.

---

### Problema 1 — A instalação não está automática

**O que acontece:** o instalador abre, mas mostra as telas de idioma/teclado como numa instalação comum.

**Causa:** o Windows não encontrou o `Autounattend.xml`. Motivos comuns: arquivo dentro de uma pasta; nome diferente (ex.: `autounattend (1).xml`); extensão oculta escondendo um `.txt` no final; ISO reconstruída sem o arquivo.

**Como confirmar:** conecte o pendrive em outro computador, ative `Exibir → Mostrar → Extensões de nomes de arquivo` e verifique se existe exatamente `Autounattend.xml` no primeiro nível, ao lado de `boot`, `efi` e `sources`.

**Solução:**
1. Apague cópias em pastas ou com nome errado;
2. Copie novamente o arquivo original para a raiz;
3. Garanta que a extensão final é `.xml`;
4. Reinicie a instalação do zero.

**Como evitar:** exiba as extensões antes de copiar; nunca renomeie o arquivo; siga a [lista de verificação](#lista-de-verificação-evita-90-dos-problemas).

**Validação:** a instalação vai direto para "Instalando o Windows", sem perguntar idioma.

---

### Problema 2 — Erro de chave de produto

**O que acontece:** logo no início, o erro "A chave de produto digitada não corresponde a nenhuma imagem do Windows" interrompe a instalação.

**Causa:** a ISO não contém a edição **Pro** (ex.: ISO somente Home ou modificada). A chave do arquivo apenas seleciona a edição; se ela não existe na imagem, o instalador falha.

**Como confirmar:** verifique a origem da ISO. ISOs oficiais da Microsoft são multi-edição e contêm a Pro; ISOs de recovery de fabricantes ou modificadas podem não conter.

**Solução (escolha uma):**
* **Recomendada:** baixe a ISO oficial no site da Microsoft e refaça a mídia;
* **Instalar Home:** troque a chave no XML por `YTMG3-N6DKC-DKB77-7M9GH-8HVX7` ([seção 5](#5-personalizações-comuns));
* **Deixar o instalador decidir:** apague o bloco de `<ProductKey>` até `</ProductKey>` — mas, se a ISO tiver várias edições, o instalador voltará a perguntar qual instalar.

**Como evitar:** use sempre a ISO oficial da Microsoft.

**Validação:** a instalação começa a copiar arquivos sem exibir tela de chave/edição.

---

### Problema 3 — Erro "O Windows não pode processar o arquivo de resposta"

**O que acontece:** mensagem citando o "arquivo de resposta autônoma" (unattend answer file), às vezes indicando o passe (`windowsPE`, `specialize`…).

**Causa:** o XML foi corrompido ao ser editado — Word/WordPad trocando aspas e codificação, um `<` ou `>` apagado sem querer, ou salvamento em codificação errada.

**Como confirmar:** se você editou o arquivo antes do erro, essa é quase certamente a causa. Arraste o arquivo para o Chrome/Edge: se estiver quebrado, o navegador mostra o erro e a linha.

**Solução:**
1. Descarte a cópia editada e baixe novamente o arquivo original;
2. Se precisar editar, siga as regras da [seção 5](#5-personalizações-comuns) (Bloco de Notas, só valores, salvar em UTF-8);
3. Valide no navegador antes de usar;
4. Copie para a mídia e reinstale.

**Como evitar:** nunca editar com Word; alterar apenas valores; validar no navegador após qualquer edição.

**Validação:** a instalação prossegue sem mensagens sobre o arquivo de resposta.

---

### Problema 4 — A instalação recomeça em loop após reiniciar

**O que acontece:** o instalador copia os arquivos, reinicia e começa tudo de novo, indefinidamente.

**Causa:** o computador continua inicializando pelo pendrive/ISO. Como o arquivo apaga o disco automaticamente, cada ciclo reinstala.

**Como confirmar:** após a reinicialização, a tela azul inicial de instalação aparece de novo.

**Solução:**
1. Remova o pendrive (ou desconecte a ISO da VM);
2. Se o loop já ocorreu, desligue segurando o botão de energia;
3. Ligue novamente — o computador seguirá pelo disco instalado.

**Como evitar:** fique por perto na primeira reinicialização para remover a mídia; em VMs, deixe o disco antes do CD/DVD na ordem de boot.

**Validação:** após reiniciar, aparece "Preparando o Windows" em vez da tela de instalação.

---

### Problema 5 — O disco não aparece para o instalador

**O que acontece:** erro "Não foi possível encontrar unidades" ou falha ao criar/formatar partições.

**Causa:** o instalador não tem driver para o controlador de armazenamento. Casos típicos: VM **QEMU/KVM/Proxmox com disco VirtIO**; notebooks Intel recentes com modo **RAID/Intel RST/VMD** na BIOS; controladoras RAID de servidor.

**Como confirmar:** na instalação, pressione `Shift + F10` e digite:

```
diskpart
list disk
```

Se nenhum disco aparecer, é falta de driver. Se o disco aparece, o problema é outro ([Problema 7](#problema-7--disco-errado-foi-formatado-ou-medo-de-formatar-o-errado)).

**Solução:**
* **QEMU/KVM/Proxmox:** desligue a VM e troque o disco de VirtIO para **SATA** (mantém a automação completa);
* **Notebooks Intel:** na BIOS (`F2`/`DEL` ao ligar), mude o modo de armazenamento de `RAID/RST/VMD` para **AHCI**, salve e reinstale. Alternativa avançada: baixar o driver Intel RST no site do fabricante, extrair para uma pasta do pendrive e carregar manualmente na tela de discos.

**Como evitar:** crie discos de VM como SATA; confira o modo AHCI em notebooks novos antes de instalar.

**Validação:** `diskpart` → `list disk` mostra o disco, e a instalação avança para a cópia de arquivos.

---

### Problema 6 — A tela de requisitos do Windows 11 aparece mesmo assim

**O que acontece:** apesar do bypass, aparece "Este computador não pode executar o Windows 11".

**Causa:**
* O XML não foi lido ([Problema 1](#problema-1--a-instalação-não-está-automática)) — sem ele, os bypasses nunca são aplicados;
* O arquivo foi editado e a seção de bypasses (`RunSynchronous`) foi removida/quebrada;
* **Windows 11 24H2+ em CPU anterior a ~2009** (sem SSE4.2): limite físico, sem solução por software — nesse caso o sintoma costuma ser travamento, não a tela de requisitos.

**Como confirmar:** se a instalação também pediu idioma no início, o XML não foi lido. Se estava automática até essa tela, a seção de bypasses foi alterada.

**Solução:**
1. Restaure o `Autounattend.xml` original na raiz da mídia;
2. Reinstale do zero;
3. CPU anterior a ~2009: use o Windows 11 23H2 ou o Windows 10.

**Como evitar:** não editar a seção de bypasses; conferir a idade da CPU antes do 24H2.

**Validação:** a instalação passa direto para a cópia de arquivos.

---

### Problema 7 — Disco errado foi formatado (ou medo de formatar o errado)

**O que acontece:** em computadores com mais de um disco, o arquivo sempre apaga o **Disco 0** — que nem sempre é o disco que você imagina (a numeração segue portas/slots, não tamanhos).

**Causa:** `<DiskID>0</DiskID>` fixo no arquivo, por previsibilidade.

**Como confirmar (ANTES de instalar):** na primeira tela da instalação, pressione `Shift + F10` e digite:

```
diskpart
list disk
```

Identifique pelo **tamanho** qual número corresponde a cada disco. Se o destino não for o Disco 0, **não prossiga** sem uma das soluções abaixo.

**Solução (escolha uma):**
* **Mais segura (recomendada):** desligue e **desconecte fisicamente todos os discos, exceto o de destino** — com um único disco, ele será o Disco 0. Reconecte os demais após instalar;
* **Avançada:** altere as **duas** ocorrências de `<DiskID>0</DiskID>` no XML para o número correto.

**Se o disco errado já foi apagado:** pare tudo imediatamente e não grave nada no disco. Ferramentas ou serviços de recuperação de dados podem resgatar parte dos arquivos; as chances caem a cada gravação.

**Como evitar:** backup sempre; desconectar discos de dados antes de instalar; conferir com `diskpart` na dúvida.

**Validação:** após instalar, abra `Gerenciamento de Disco` e confirme os discos de dados intactos e o Windows no disco pretendido.

---

### Problema 8 — O computador não inicia após a instalação

**O que acontece:** "No bootable device" / "Boot device not found" ao ligar.

**Causa:** modo **Legacy BIOS/CSM** ativo (incompatível com o particionamento UEFI/GPT do arquivo), ou a entrada "Windows Boot Manager" não é a primeira na ordem de boot.

**Como confirmar:** entre na BIOS/UEFI (`DEL`, `F2`, `F10` ou `Esc` ao ligar) e veja o "Boot Mode": se estiver em `Legacy`/`CSM`, é essa a causa.

**Solução:**
1. Na BIOS/UEFI, altere `Boot Mode` para **UEFI Only** (ou desative o CSM);
2. Salve (`F10`) e saia;
3. **Refaça a instalação do início** (a anterior foi feita para o modo errado), escolhendo no menu de boot a entrada do pendrive com prefixo **UEFI**;
4. Se o modo já era UEFI: mova "Windows Boot Manager" para a primeira posição da ordem de boot.

**Como evitar:** confirmar o modo UEFI antes de instalar; em VMs, usar as configurações da [seção 4](#4-instalação-em-máquinas-virtuais).

**Validação:** o computador inicia sem mídia conectada e o `msinfo32` mostra "Modo da BIOS: UEFI".

---

### Problema 9 — VirtualBox não automatiza ou não dá boot

**O que acontece:** a instalação pede cliques, ou a VM nem inicia pela ISO.

**Causa:** o recurso "Unattended Installation" do próprio VirtualBox foi usado (injeta um arquivo de resposta próprio que conflita); a VM foi criada sem **EFI**; ou a ISO usada é a original, sem o `Autounattend.xml` dentro.

**Como confirmar:** veja em `Configurações → Sistema → Placa-mãe` se "Habilitar EFI" está marcado; lembre se você pulou a Unattended Installation na criação.

**Solução:**
1. Exclua a VM problemática (mais rápido que consertar);
2. Recrie seguindo a tabela do [VirtualBox na seção 4](#virtualbox-gratuito);
3. Conecte a ISO **editada** (seção 3.2) ou a original + segunda ISO só com o XML;
4. Inicie.

**Como evitar:** sempre pular a Unattended Installation e conferir o EFI.

**Validação:** a VM progride sem pedir cliques até a Área de Trabalho.

---

### Problema 10 — VMware: Easy Install quebra a automação

**O que acontece:** a VMware pede chave/usuário na criação da VM, ou a instalação sai com configurações diferentes das deste projeto.

**Causa:** o **Easy Install** foi ativado (acontece quando a ISO é apontada durante o assistente de criação). Ele injeta um disquete virtual (`autoinst.flp`) com arquivo de resposta próprio, que tem prioridade.

**Como confirmar:** presença de um dispositivo **Floppy** com `autoinst.flp` nas configurações da VM.

**Solução:**
1. Crie a VM escolhendo **"I will install the operating system later"**;
2. Conecte a ISO editada só depois, nas configurações;
3. Para aproveitar a VM existente: remova o Floppy `autoinst.flp` e conecte a ISO correta;
4. Inicie.

**Como evitar:** nunca apontar a ISO durante o assistente de criação da VMware.

**Validação:** sem Floppy nas configurações; a instalação cria a conta `UserLocal` em pt-BR conforme este README.

---

### Problema 11 — Hyper-V falha no início ou não inicializa

**O que acontece:** a VM não dá boot na ISO ou a instalação falha logo no começo.

**Causa:** VM criada como **Geração 1** (BIOS Legacy); **Memória Dinâmica** com RAM inicial muito baixa (o instalador falha por falta de memória antes de o bypass agir); ou ordem de boot sem o DVD primeiro.

**Como confirmar:** no Gerenciador do Hyper-V, veja "Geração" no resumo da VM e a RAM inicial em `Configurações → Memória`.

**Solução:**
1. Geração 1: exclua e recrie como **Geração 2** (não há conversão);
2. Defina RAM inicial de **4096 MB** (pode reduzir após instalar);
3. Em `Configurações → Firmware`, coloque o DVD antes do disco;
4. Inicie e pressione uma tecla se aparecer "Press any key to boot from CD".

**Como evitar:** para este projeto, sempre Geração 2 com 4 GB iniciais.

**Validação:** a VM inicializa pela ISO e conclui sem intervenção.

---

### Problema 12 — O Windows pede conta Microsoft mesmo assim

**O que acontece:** a instalação corre automatizada, mas ao final aparecem telas pedindo internet ou conta Microsoft.

**Causa:** o passe final (`oobeSystem`) não foi aplicado — em geral porque o XML foi editado e a seção `<UserAccounts>` ou `<OOBE>` foi removida/quebrada. (Se a instalação inteira pediu cliques desde o início, o caso é o [Problema 1](#problema-1--a-instalação-não-está-automática).)

**Como confirmar:** se as fases anteriores foram automáticas, o XML foi lido; o problema está em edição da parte final do arquivo.

**Solução:**
1. **Para concluir agora sem conta Microsoft** (Windows 11): pressione `Shift + F10`, digite `start ms-cxh:localonly` e Enter — abre o assistente de conta local;
2. Para as próximas instalações, restaure o XML original e refaça a mídia.

**Como evitar:** não editar as seções `<OOBE>` e `<UserAccounts>`; validar o XML no navegador após edições.

**Validação:** a instalação termina direto na Área de Trabalho, logada como `UserLocal`.

---

### Problema 13 — O login automático não funciona

**O que acontece:** o Windows instala, mas para na tela de login.

**Causa:** edição que alterou o nome da conta em um lugar e não no outro — `<AutoLogon><Username>` precisa ser idêntico a `<LocalAccount><Name>` — ou senha adicionada à conta sem atualizar o bloco de AutoLogon.

**Como confirmar:** a conta aparece na tela de login, mas não entra sozinha. Compare os dois nomes no XML.

**Solução (sem reinstalar):**
1. Entre manualmente (a conta padrão não tem senha — clique em Entrar);
2. `Win + R` → `netplwiz` → desmarque "Os usuários devem digitar um nome de usuário e senha…" → Aplicar → informe a senha (em branco, se não houver);
3. Se a opção não aparecer (Windows 11 com Windows Hello): `Configurações → Contas → Opções de entrada` → desative "Para maior segurança, permita a entrada do Windows Hello…" e repita o passo 2;
4. Reinicie para testar.

**Como evitar:** ao personalizar o nome do usuário, alterá-lo em **todas** as ocorrências ([seção 5](#5-personalizações-comuns)).

**Validação:** duas reinicializações seguidas entrando sozinho na Área de Trabalho.

---

### Problema 14 — Teclado digitando símbolos errados

**O que acontece:** `ç`, `?` ou acentos saem trocados.

**Causa:** o teclado físico não é ABNT2 (padrão configurado). Notebooks importados usam layout americano (US); teclados bem antigos podem ser ABNT de 1ª geração.

**Como confirmar:** olhe o teclado físico — sem tecla `Ç` dedicada, é layout internacional (US).

**Solução (sem reinstalar):**
1. `Configurações → Hora e idioma → Idioma e região`;
2. Três pontos ao lado de "Português (Brasil)" → `Opções de idioma`;
3. Em Teclados: `Adicionar um teclado` → `Estados Unidos (Internacional)` (teclado US) ou `Português (Brasil ABNT)`;
4. Remova o teclado que não corresponde ao físico e teste.

Para já instalar com o layout certo, ajuste o XML conforme a [seção 5](#5-personalizações-comuns).

**Como evitar:** conferir o teclado físico antes e ajustar o XML se não for ABNT2.

**Validação:** todas as teclas produzem os caracteres impressos nelas.

---

### Problema 15 — Sem internet ou drivers faltando

**O que acontece:** sem Wi-Fi/rede após instalar, ou itens com alerta amarelo no Gerenciador de Dispositivos.

**Causa:** o Windows não traz driver nativo para todos os dispositivos (comum em Wi-Fi recente, chipsets novos e rede 2.5G). Este projeto instala 100% offline e não injeta drivers.

**Como confirmar:** clique direito no Iniciar → `Gerenciador de Dispositivos` → procure triângulos amarelos.

**Solução:**
1. **Com rede cabeada disponível:** conecte o cabo e rode `Configurações → Windows Update → Verificar atualizações`, incluindo `Opções avançadas → Atualizações opcionais → Atualizações de driver`;
2. **Sem nenhuma rede:** em outro computador, baixe o driver de **rede/Wi-Fi** do seu modelo exato no site do fabricante, leve por pendrive e instale; depois o Windows Update cuida do resto;
3. **Em VMs:** instale VirtualBox Guest Additions, VMware Tools ou (QEMU/Proxmox) os drivers da ISO `virtio-win`. No Hyper-V os drivers são nativos.

**Como evitar:** baixar o driver de rede do modelo antes de formatar; em VMs, ter a ISO de complementos à mão.

**Validação:** Gerenciador de Dispositivos sem alertas e internet funcionando.

---

### Problema 16 — Erro durante o passe "specialize"

**O que acontece:** após a primeira reinicialização, erro como "O Windows não pôde concluir a instalação" ou "não foi possível aplicar as configurações autônomas", citando o passe `specialize`.

**Causa:** valor inválido introduzido por edição — os clássicos são fuso horário digitado errado (o nome precisa ser exatamente um fuso reconhecido pelo Windows) ou nome de computador inválido (mais de 15 caracteres, espaços ou símbolos).

**Como confirmar:** se você alterou `<TimeZone>` ou `<ComputerName>`, é quase certamente isso. O log confirma: `Shift + F10` → `notepad C:\Windows\Panther\setuperr.log`.

**Solução:**
1. Restaure o XML original, ou corrija o valor (fuso exato via `tzutil /l`; nome com até 15 letras/números/hífens, ou apenas `*` para automático);
2. Refaça a mídia e **reinstale do zero** — erros no specialize deixam a instalação inconsistente.

**Como evitar:** copiar o nome do fuso do `tzutil /l`; manter `*` no nome do computador sem necessidade específica.

**Validação:** a instalação atravessa as reinicializações sem erros até a Área de Trabalho.

---

### Problema 17 — Tela preta ou travamento no primeiro boot

**O que acontece:** a instalação parece concluir, mas o primeiro boot fica em tela preta ou congela.

**Causa:** em VMs, aceleração 3D com driver de vídeo problemático ou memória de vídeo baixa; em máquinas físicas, driver de vídeo genérico em GPUs muito novas/antigas — ou o sistema está apenas lento finalizando.

**Como confirmar:** aguarde 15 minutos antes de concluir que travou (mouse se mexendo = sistema vivo). Em VM, teste desligar a aceleração 3D.

**Solução:**
1. **VirtualBox:** VM desligada → `Configurações → Monitor` → desmarque "Habilitar aceleração 3D" e defina 128 MB de vídeo → ligue;
2. **Máquina física:** force o desligamento e ligue 2 vezes seguidas para acionar o reparo automático → `Opções avançadas → Configurações de inicialização → Modo de segurança com rede` → instale o driver de vídeo do fabricante → reinicie;
3. Persistindo, reinstale com a mídia refeita (descarta corrupção de cópia).

**Como evitar:** em VMs, instalar com 3D desligado (ativar depois); em físico, ter o driver de vídeo baixado antes.

**Validação:** dois boots seguidos chegando à Área de Trabalho com resolução correta.

---

### Problema 18 — Disco pequeno demais

**O que acontece:** falha na fase de disco ou logo depois, por falta de espaço.

**Causa:** o bypass de armazenamento permite tentar em discos abaixo do mínimo oficial (64 GB), mas abaixo de ~25–30 GB a instalação falha ou o sistema fica inutilizável após as primeiras atualizações.

**Como confirmar:** verifique o tamanho do disco/VM (alguns assistentes criam discos virtuais de 50 GB ou menos por padrão).

**Solução:** aumente o disco para **64 GB ou mais** (recriando ou expandindo o disco virtual) e reinstale; em físico, use disco maior.

**Como evitar:** padronizar 64 GB+ em qualquer instalação com este projeto.

**Validação:** instalação concluída e `C:` com pelo menos ~20 GB livres.

---

### Problema 19 — O Rufus criou configurações próprias

**O que acontece:** a instalação foi automática, mas o resultado não bate com este README (outro usuário, outro idioma, sem os ajustes de privacidade).

**Causa:** as opções da janela "Windows User Experience" do Rufus ficaram marcadas na gravação — o Rufus então gera um arquivo de resposta próprio que conflita com o deste projeto.

**Como confirmar:** o usuário criado não é `UserLocal`, ou as configurações divergem das descritas aqui.

**Solução:**
1. Regrave o pendrive **desmarcando todas** as opções da janela de personalizações;
2. Copie o `Autounattend.xml` deste projeto para a raiz;
3. Reinstale.

**Como evitar:** sempre desmarcar as personalizações do Rufus ([seção 3.1](#31-preparar-a-mídia--pendrive-computador-físico)).

**Validação:** a instalação cria a conta `UserLocal` em pt-BR com teclado ABNT2.

---

### Problema 20 — Investigando qualquer outro erro (logs)

Se o seu problema não está listado, os registros do instalador dizem exatamente o que falhou:

1. Na tela de erro, pressione `Shift + F10` para abrir o Prompt de Comando;
2. Abra os logs (um dos dois caminhos existirá, conforme a fase do erro):

   ```
   notepad X:\Windows\Panther\setuperr.log
   notepad C:\Windows\Panther\setuperr.log
   ```

   `setuperr.log` contém apenas os erros; `setupact.log` (mesma pasta) contém o passo a passo completo;
3. Localize as últimas linhas com "Error" — elas apontam a configuração ou componente que falhou;
4. Com a mensagem exata em mãos, pesquise-a ou abra um issue no repositório anexando o trecho do log.

---

## 8. Como funciona (detalhes técnicos)

*Leitura opcional — você não precisa desta seção para usar o projeto.*

### As três fases da instalação

| Fase | Nome técnico | O que o arquivo faz |
| --- | --- | --- |
| 1 | `windowsPE` | Define idioma/teclado do instalador, aplica os bypasses do Win11, apaga e particiona o Disco 0, aceita a EULA e seleciona a edição Pro |
| 2 | `specialize` | Gera nome de computador automático, define fuso horário de Brasília e idioma pt-BR do sistema |
| 3 | `oobeSystem` | Oculta todas as telas de configuração inicial, cria a conta `UserLocal` (admin, sem senha), ativa o login automático e aplica os ajustes de privacidade |

### Fluxo completo

```mermaid
flowchart TD
    Start([Ligar o computador]) --> FirmwareCheck{Tipo de firmware?}
    FirmwareCheck -- BIOS Legacy --> Abort[FALHA: configuração padrão exige UEFI]
    FirmwareCheck -- UEFI --> BootEFI[Inicializa pela mídia de instalação]

    BootEFI --> LoadWinPE[Carrega o ambiente de instalação]
    LoadWinPE --> SearchXML{Autounattend.xml na raiz?}

    SearchXML -- Não --> ManualInstall[Instalação manual tradicional]
    SearchXML -- Sim --> Pass1[Fase 1: windowsPE]

    Pass1 --> Bypass[Aplica bypasses do Windows 11]
    Bypass --> Wipe[Apaga e particiona o Disco 0]
    Wipe --> Copy[Copia os arquivos do Windows]

    Copy --> R1((Reinicialização))
    R1 --> Pass2[Fase 2: specialize]
    Pass2 --> R2((Reinicialização))

    R2 --> Pass3[Fase 3: oobeSystem]
    Pass3 --> User[Cria conta UserLocal + login automático]
    User --> Priv[Aplica ajustes de privacidade]
    Priv --> Desktop([Área de Trabalho pronta])
```

### Particionamento aplicado (UEFI/GPT)

| Ordem | Partição | Tamanho | Sistema de arquivos | Rótulo | Letra |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | EFI (System) | 300 MB | FAT32 | `System` | — |
| 2 | MSR (Reservada) | 16 MB | — | — | — |
| 3 | Windows | Restante do disco | NTFS | `Windows` | `C` |

O ambiente de recuperação (WinRE) fica em `C:\Recovery` — comportamento normal e funcional; partição de recuperação separada é opcional.

### Bypasses do Windows 11 aplicados

| Requisito ignorado | Chave de registro (`LabConfig`) |
| --- | --- |
| TPM 2.0 | `BypassTPMCheck` |
| Secure Boot | `BypassSecureBootCheck` |
| Processador | `BypassCPUCheck` |
| Memória RAM | `BypassRAMCheck` |
| Armazenamento | `BypassStorageCheck` |

### Ajustes de privacidade no primeiro login

```text
AllowCortana = 0                     desativa a Cortana
AllowTelemetry = 0                   telemetria no mínimo permitido pela edição
DisableWindowsConsumerFeatures = 1   remove apps e sugestões promocionais
DisableSoftLanding = 1               desativa dicas e Windows Spotlight
```

> Observação honesta: no Windows Pro, o nível de telemetria 0 é elevado automaticamente para 1 pela Microsoft; o valor 0 só é totalmente respeitado nas edições Enterprise/Education.

---

## 9. Compatibilidade

| Ambiente | Suporte | Observações |
| --- | --- | --- |
| Windows 10 x64 | ✅ | Qualquer ISO oficial com a edição Pro |
| Windows 11 x64 (até 23H2) | ✅ | Bypasses cobrem todos os requisitos |
| Windows 11 24H2 ou mais novo | ✅* | *Exceto CPUs anteriores a ~2009 (limite físico) |
| Windows 32 bits (x86) | ❌ | O arquivo é específico para 64 bits (amd64) |
| Computadores físicos UEFI | ✅ | Configuração padrão |
| Computadores Legacy BIOS/MBR | ⚙️ | Bloco alternativo comentado no XML |
| VirtualBox | ✅ | EFI ligado + pular Unattended Installation |
| VMware Workstation/Player | ✅ | Sem Easy Install ("install the OS later") |
| Hyper-V | ✅ | Somente Geração 2; RAM inicial ≥ 4 GB |
| QEMU / KVM / Proxmox | ✅ | Firmware OVMF; disco SATA (VirtIO exige drivers) |

---

## 10. Estrutura do projeto e licença

```
Autounattend.xml   → arquivo que automatiza a instalação (arquivo principal)
README.md          → esta documentação
LICENSE            → licença MIT
```

Distribuído sob a licença **MIT** — uso, cópia e modificação livres (consulte o arquivo `LICENSE`).

As chaves de produto citadas nesta documentação são **chaves genéricas públicas (GVLK)** documentadas pela própria Microsoft, usadas apenas para seleção de edição; elas **não ativam o Windows**. A licença/ativação do Windows é responsabilidade do usuário.
