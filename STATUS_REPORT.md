# STATUS REPORT — powerbi-project-portfolio-management

**Data:** 2026-04-18
**Executor:** Claude Code (instância do projeto powerbi-project-portfolio-management)
**Origem:** /home/vitormrodovalho/Documents/powerbi-project-portfolio-management
**Destino:** /home/vitormrodovalho/projects/powerbi-project-portfolio-management

## 1. Estado pré-migração
- Branch: main
- Commit HEAD: 11c6f548f275b783fe17556036f8f7a0bc523d69 ("Initial commit")
- Dirty: 0 arquivos
- Unpushed: 0 commits (remote `origin/main` em sincronia com HEAD local)
- Tamanho: 4.7 MB

## 2. Git — ações executadas
- Commits criados: nenhum
- Push: não necessário — remote já em sincronia
- Branches empurradas: nenhuma

## 3. Dados externos (map-deps)
Referências encontradas a caminhos fora do projeto:

| Arquivo do projeto | Referência externa | Status |
|---|---|---|
| _(nenhuma)_ | _(n/a)_ | Projeto não referencia `pbix-extraction/`, `Backup/pbix-extracted/`, `Backup/pbix-new/`, nem nenhum caminho absoluto em `/home/vitormrodovalho/` ou `~/Downloads/` |

Comando: `rg -l -e 'pbix-extraction' -e 'pbix-extracted' -e 'pbix-new' -e 'Documents/Backup' -e 'OneDrive_2026' -e '/home/vitormrodovalho/' .` → 0 matches.

## 4. Mineração de dados (se aplicável)
- Status: N/A do ponto de vista deste projeto — não há código/scripts apontando para as pastas brutas de `pbix-*`
- Fontes consumidas: indeterminado por inspeção apenas desta origem. Se os dashboards deste repo foram produzidos a partir de `pbix-extraction/` / `pbix-new/`, a conexão não está documentada aqui
- Pendências: nenhuma do ponto de vista deste projeto

## 5. Migração — integridade
- `cp -a` executado: sim
- `diff -qr` exit code: 0 (`/tmp/migration-diff-powerbi-project-portfolio-management.txt` 0 linhas)
- Verificação git pós-cópia: HEAD bate (`11c6f54…`)
- Origem deletada: sim

## 6. Sinalizações para consolidação (fase 3)

**Fora de escopo — decisão da conversa mestra**: as pastas abaixo existem no filesystem mas **não** são referenciadas por este projeto, portanto não decido por elas:

- `~/Documents/pbix-extraction/` (1.2 GB) — provável fonte bruta comum aos 4 projetos `powerbi-*`; precisa análise cruzada após as 4 migrações
- `~/Documents/Backup/pbix-extracted/` (2.3 MB)
- `~/Documents/Backup/pbix-new/` (2.5 GB)

Recomendação: consolidar a decisão apenas depois que os 4 reports (`powerbi-construction-analytics`, `powerbi-financial-analytics`, `powerbi-project-portfolio-management`, `powerbi-real-estate-bpo`) concordarem que nenhum deles depende dessas pastas. Se nenhum referencia, as pastas são candidatas a delete — mas a decisão não é deste projeto isolado.

Arquivos externos que PODEM ser deletados com segurança após esta migração (escopo deste projeto):
- _(nenhum)_

Arquivos externos que NÃO PODEM ser deletados ainda:
- _(nenhum decidido aqui)_

## 7. Problemas encontrados
Nenhum.

## 8. Confirmação final
- [x] Projeto funcional em `~/projects/powerbi-project-portfolio-management/`
- [x] Git remoto em dia
- [x] Origem removida
- [x] Report pronto para envio à conversa mestra
