# Relatório de Revisão — Projeto Autounattend.xml

**Escopo:** revisão completa dos 3 arquivos do projeto (`Autounattend.xml`, `README.md`, `LICENSE`), validação da lógica de instalação, avaliação da documentação sob a ótica de usuários não técnicos e análise de cenários em ambientes físicos e virtualizados.

---

## 1. Resumo executivo

O projeto está **bem construído e funcional**. O XML é sintaticamente válido (verificado com parser e `xmllint`), usa UTF-8 sem BOM, cobre os três passes corretos (`windowsPE` → `specialize` → `oobeSystem`) e adota a abordagem tecnicamente correta para dispensar a conta Microsoft (conta local provisionada + `HideOnlineAccountScreens`, em vez do frágil truque BypassNRO). O particionamento UEFI/GPT segue o layout recomendado pela Microsoft e a lógica geral de instalação é coerente e reproduzível.

Foram identificados **2 problemas de conteúdo no XML** (um comentário com instrução truncada e o uso de opções depreciadas), **inconsistências menores de documentação interna**, e — o ponto mais relevante — **lacunas significativas na seção de solução de problemas do README**, que cobria apenas 5 cenários e não tratava de virtualização, drivers de armazenamento, corrupção do XML, erros do passe specialize, autologon, teclado, rede, entre outros. Nenhum achado é impeditivo; todos foram corrigidos nos arquivos revisados entregues junto a este relatório.

---

## 2. Achados no `Autounattend.xml`

### 2.1 Críticos / funcionais

| # | Achado | Severidade | Situação |
| - | ------ | ---------- | -------- |
| A1 | **Comentário do cabeçalho com instrução truncada** (linhas 9–10): "Para BIOS Legacy/MBR ou deteccao automatica de firmware." — a frase termina sem dizer o que fazer. O usuário que precisa de BIOS/MBR fica sem orientação, e o cabeçalho promete algo que o arquivo não entrega. | Média | **Corrigido**: frase completada e adicionado bloco alternativo BIOS/MBR comentado ao final do arquivo, com passo a passo de ativação. |
| A2 | **`SkipMachineOOBE` e `SkipUserOOBE`** — opções **depreciadas pela Microsoft desde o Windows 8**. São redundantes aqui (as opções `Hide*` + a conta provisionada já suprimem todas as telas) e podem gerar comportamento imprevisível em builds recentes do Windows 11 (há relatos de OOBE incompleto). | Média | **Corrigido**: removidas, com nota explicativa no comentário. |
| A3 | **`NetworkLocation`** — ignorado desde o Windows 8 (não tem efeito). Não causa erro, mas polui o arquivo e sugere um efeito inexistente. | Baixa | **Corrigido**: removido. |

### 2.2 Pontos validados como corretos (sem alteração)

* **Passes e componentes**: `windowsPE` (International-Core-WinPE + Setup), `specialize` (Shell-Setup + International-Core), `oobeSystem` (Shell-Setup) — estrutura canônica, arquiteturas `amd64` e tokens corretos.
* **Particionamento**: EFI 300 MB FAT32 + MSR 16 MB + Primary NTFS com `Extend` — layout válido; 300 MB de EFI é acima do mínimo (100 MB) e adequado. `ImageInstall` apontando para a partição 3 é consistente.
* **Ausência de partição de recuperação dedicada**: aceitável — o WinRE fica em `C:\Recovery` (funcional). Foi apenas documentado no XML e no README para transparência.
* **Bypasses LabConfig**: chaves, caminho de registro e momento de execução (RunSynchronous no windowsPE, antes da checagem) corretos. Inofensivos no Windows 10.
* **Remoção do BypassNRO**: decisão correta e bem justificada no comentário (estava no caminho errado e é desnecessário com conta provisionada).
* **GVLK `W269N-…`**: chave genérica de seleção de edição Pro, válida para Win10/11; comentário deixa claro que não ativa. Adicionada ao comentário a GVLK do Home como alternativa.
* **AutoLogon sem `LogonCount`**: correto para logon automático **permanente** — sem contador, o Windows mantém `AutoAdminLogon=1` sem expiração. Comportamento agora documentado em comentário.
* **`ProtectYourPC=3`**: presente e obrigatório quando o OOBE é suprimido. OK.
* **FirstLogonCommands**: executam elevados (conta admin no primeiro logon), então os `reg add` em HKLM funcionam. A nota sobre telemetria (nível 0 → elevado a 1 no Pro) é honesta e correta.
* **Encoding**: UTF-8 sem BOM, quebras LF, XML declarado corretamente — compatível com o Setup.

### 2.3 Inconsistências de documentação interna

* Comentários do XML referiam-se à conta como **"User"** (linhas 16 e no cabeçalho do passe 3), mas a conta real é **"UserLocal"**. Divergência corrigida em todas as ocorrências.
* Ausência de aviso de segurança sobre conta admin **sem senha + autologon permanente**. Adicionado aviso no cabeçalho do XML e destaque no README.

### 2.4 Fragilidades estruturais documentadas (decisões de design, não defeitos)

* **`DiskID` fixo em 0**: previsível, porém perigoso em máquinas multi-disco (a numeração segue portas, não tamanhos). Mantido por simplicidade; risco agora tratado em detalhe no README (item 7) com procedimento de verificação via `diskpart` **antes** de instalar.
* **Sem injeção de drivers** (não há componente `Microsoft-Windows-PnpCustomizationsWinPE`): em QEMU/KVM/Proxmox com disco VirtIO e em notebooks Intel com VMD/RST, o instalador não enxerga o disco. Solução simples (disco SATA / modo AHCI) documentada no README (item 5). Injeção de drivers foi avaliada e **não** incorporada por padrão para manter o projeto de arquivo único e simples — registrada como melhoria futura opcional.
* **Windows 11 24H2 em CPUs pré-SSE4.2 (~2009)**: limite físico sem bypass possível. Documentado no XML e no README.

---

## 3. Achados no `README.md` original

### 3.1 Problemas de conteúdo

| # | Achado | Situação |
| - | ------ | -------- |
| R1 | Badge **"PowerShell 5.1+"** — o projeto não contém nenhum script PowerShell; badge enganosa. | Removida. |
| R2 | Typo no diagrama de sequência: "Sistema Operacional Operacional". | Diagrama simplificado/corrigido na revisão. |
| R3 | Diagrama de fluxo usava jargão sem tradução para leigos ("RAMDisk", "Passe 4/7", "ApplyWIM"). | Reescrito com rótulos compreensíveis, mantendo a precisão. |
| R4 | O aviso Legacy BIOS dizia que "precisam de ajustes" **sem dizer quais** — beco sem saída para o usuário não técnico. | Agora aponta para o bloco alternativo comentado no próprio XML. |
| R5 | Chave GVLK apresentada sem explicar com clareza para leigos que é pública/oficial e não pirataria nem ativação. | Explicação ampliada, incluindo alternativa Home. |
| R6 | Nenhuma menção ao risco da conta sem senha. | Aviso dedicado adicionado. |

### 3.2 Lacunas na solução de problemas (o achado principal)

O README original cobria **5 cenários** (XML não encontrado, chave de produto, loop de boot, disco errado, Legacy BIOS) — e nenhum deles no formato completo pedido (faltavam "como identificar", "como evitar" e "validação após correção"). Cenários inteiros estavam ausentes, incluindo **todos os de virtualização**.

O README revisado passa a cobrir **20 cenários**, cada um com a estrutura de 6 campos solicitada (descrição → causa → identificação → solução passo a passo → prevenção → validação):

1. XML não encontrado (instalação vira manual)
2. Chave de produto × ISO sem edição Pro
3. **XML corrompido por edição** ("não pode analisar o arquivo de resposta") — novo
4. Loop de instalação após reinicialização
5. **Disco invisível / drivers de armazenamento (VirtIO, Intel VMD/RST, RAID)** — novo
6. **Tela de requisitos do Win11 aparece mesmo assim (inclui limite 24H2/CPU antiga)** — novo
7. Disco errado formatado (agora com prevenção via `diskpart` e recuperação)
8. "No bootable device" / Legacy BIOS (agora com validação via `msinfo32`)
9. **VirtualBox: EFI + conflito com Unattended Installation nativa** — novo
10. **VMware: conflito com Easy Install (autoinst.flp)** — novo
11. **Hyper-V: Geração 1 × 2, memória dinâmica insuficiente** — novo
12. **OOBE pede conta Microsoft mesmo assim (inclui saída de emergência `ms-cxh:localonly`)** — novo
13. **Autologon não funciona (netplwiz / Windows Hello)** — novo
14. **Teclado errado (ABNT2 × US × ABNT)** — novo
15. **Sem internet / drivers ausentes pós-instalação (físico e VMs)** — novo
16. **Erro no passe specialize (TimeZone/ComputerName inválidos + leitura de logs)** — novo
17. **Tela preta / travamento no primeiro boot (3D em VMs, vídeo em físico)** — novo
18. **Disco pequeno demais** — novo
19. **Conflito com personalizações do Rufus** — novo (antes só uma frase solta)
20. **Guia genérico de diagnóstico por logs (`setuperr.log` / `setupact.log`)** — novo

### 3.3 Adições para usuários não técnicos

* **Lista de verificação pré-instalação** (8 itens) que previne a maioria dos erros.
* **Guias por hypervisor** (VirtualBox, VMware, Hyper-V, QEMU/KVM/Proxmox) com os passos exatos de criação de VM compatível.
* **Dica da segunda ISO** contendo apenas o `Autounattend.xml` — evita que leigos precisem reconstruir a ISO para testar em VM (o Setup varre a raiz de todas as unidades).
* Tabela "o que você vê em cada fase" com duração esperada (10–30 min, 2–3 reinicializações), para o usuário saber quando algo saiu do normal.
* Tabela de compatibilidade expandida (incluindo o que **não** é suportado: x86/32 bits).
* Recomendação explícita de **testar primeiro em VM** antes de máquina física.

---

## 4. `LICENSE`

MIT padrão, texto íntegro, sem problemas. Única observação: falta quebra de linha ao final do arquivo (cosmético, sem impacto). Adicionada nota no README esclarecendo que as GVLKs citadas são chaves públicas de seleção de edição documentadas pela Microsoft e que a licença do Windows é responsabilidade do usuário — importante juridicamente para um projeto que distribui chaves em texto claro.

---

## 5. Validações executadas nesta revisão

| Validação | Ferramenta/método | Resultado |
| --- | --- | --- |
| Boa formação do XML original | `xmllint` + parser Python (ElementTree) | ✅ Válido |
| Boa formação do XML revisado | `xmllint` | ✅ Válido |
| Encoding e BOM | `file` + inspeção de bytes | ✅ UTF-8 sem BOM |
| Quebras de linha | contagem de CR | ✅ LF puro (aceito pelo Setup) |
| Estrutura de passes/componentes | enumeração programática | ✅ 3 passes, 5 componentes corretos |
| Consistência conta/AutoLogon | comparação `<Name>` × `<Username>` | ✅ Idênticos (`UserLocal`) |
| Consistência partição instalada | `ModifyPartition 3` × `InstallTo PartitionID 3` | ✅ Consistente |
| Referências cruzadas de comentários | grep | ⚠️ 2 divergências ("User" × "UserLocal") — corrigidas |
| Presença de opções depreciadas | grep | ⚠️ 3 encontradas — removidas |

> Limitação transparente: não é possível executar uma instalação real do Windows neste ambiente de revisão; a validação funcional baseia-se na análise estática contra o schema/comportamento documentado do Windows Setup e nos cenários conhecidos de campo. Recomenda-se o teste final em VM (roteiro incluso no README revisado) antes do uso em produção.

---

## 6. Melhorias sugeridas para o futuro (não aplicadas, opcionais)

1. **Variante com injeção de drivers**: um segundo XML de exemplo com `Microsoft-Windows-PnpCustomizationsWinPE`/`DriverPaths` apontando para uma pasta `$OEM$`/`Drivers` na mídia, para VirtIO e Intel RST sem intervenção manual.
2. **Variante "segura"**: conta com senha definida + autologon desativado, para quem vai usar em máquina de trabalho.
3. **Script de verificação da mídia** (opcional, PowerShell): checa nome/posição do XML, valida o XML e lista o conteúdo da raiz — reduziria a causa nº 1 de falhas (arquivo não encontrado).
4. **Partição de recuperação dedicada** (~1 GB) no layout GPT, para maior resiliência do WinRE com BitLocker.
5. **CI simples no repositório** (GitHub Actions com `xmllint`) para impedir merge de XML quebrado.
6. Publicar as duas variantes de teclado (`ABNT2` e `US Internacional`) como arquivos prontos, já que é a personalização mais pedida.

---

## 7. Arquivos entregues nesta revisão

| Arquivo | Descrição |
| --- | --- |
| `Autounattend.xml` | Versão revisada: comentário truncado corrigido, opções depreciadas removidas, inconsistências de nomenclatura corrigidas, bloco alternativo BIOS/MBR adicionado, avisos de segurança e limites do 24H2 documentados. Funcionalmente equivalente ao original nos cenários suportados. |
| `README.md` | Documentação reescrita e ampliada: linguagem para leigos, lista de verificação, guias por hypervisor, 20 cenários de erro no formato de 6 campos, tabela de compatibilidade expandida. |
| `RELATORIO-DE-REVISAO.md` | Este relatório. |
