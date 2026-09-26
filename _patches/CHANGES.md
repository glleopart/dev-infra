# dev-infra patch — changelog

## Aplicar
```zsh
cd ~/.dev-infra && git init -q 2>/dev/null; git add -A && git commit -qm "pre-patch snapshot" || true
patch -p1 --dry-run < ~/Downloads/dev-infra.diff   # debe aplicar limpio
patch -p1 < ~/Downloads/dev-infra.diff
chmod +x bin/*
install-agents                                     # regenera ~/.claude/agents y ~/.config/opencode/agents
opencode models anthropic | grep -E "opus|sonnet"  # confirma los ids de "# opencode-model:"
# Proyectos existentes: recopiar el script
cp ~/.dev-infra/scripts/audit_pipeline.py ~/projects/Pymetra/<repo>/scripts/
```
Si `patch` falla por diferencias locales, usa `files/` (árbol completo parcheado) y compara con `diff -ru`.
El diff se generó sobre un árbol reconstruido desde los archivos subidos (`mcp/.mcp.json.template` = `_mcp_json.template`).

## Bugs corregidos

| # | Archivo | Problema | Fix |
|---|---|---|---|
| 1 | `scripts/audit_pipeline.py` | `--api` nunca se parseaba: el modo API era inalcanzable | flag `--api` añadido |
| 2 | `scripts/audit_pipeline.py` | `docs/AUDIT_PROMPTS.md` **nunca se cargaba**: las reglas por proyecto se ignoraban | se inyecta en los 3 agentes |
| 3 | `scripts/audit_pipeline.py` | Reintentos re-auditaban el **mismo código** (recogido una vez fuera del bucle): 3× coste, scores ruidosos | `--max-iter` por defecto 1; con >1 pausa para arreglar y re-lee el código |
| 4 | `scripts/audit_pipeline.py` | Límite 40 archivos × 8k chars, con `.md`/`.txt`/lockfiles consumiendo cupo; lo omitido se evaluaba a ciegas | sin prosa ni lockfiles, 120×12k configurable (`--max-files`, `--max-file-chars`) y lista `__OMITTED_FILES__` para marcar UNREVIEWED |
| 5 | `scripts/audit_pipeline.py` | `pip install` silencioso al importar: falla en Ubuntu 24.04 (PEP 668) y altera el intérprete activo | error claro con instrucción |
| 6 | `scripts/audit_pipeline.py` | Modelo hardcodeado; CLI sin comprobar exit code (informe vacío si falla) | `AUDIT_MODEL` / `AUDIT_CLI_MODEL`; error si el CLI falla; cliente API perezoso |
| 7 | `bin/audit` | Exigía `ANTHROPIC_API_KEY` aunque el modo por defecto es el CLI, y la leía del `.env` **del proyecto** (en Pymetra = clave de la app) | clave solo con `--api`, desde `~/.secrets` o `$DEV_INFRA_DIR/.env` |
| 8 | `bin/audit` | `set -e` abortaba antes de capturar el exit code: nunca imprimía la ruta del informe tras un FAIL | `set +e` alrededor del pipeline |
| 9 | `agents/orchestrator.md` | Debía actualizar handoff/manifest sin tener `Write`/`Edit` | `Write`/`Edit` limitados a `docs/` + `Bash` solo lectura |
| 10 | `agents/orchestrator.md` | En Claude Code un subagente no puede lanzar subagentes: "orchestrator → @builder" no funcionaba invocado como subagente | nota de runtime: sesión principal = orchestrator (vía CLAUDE.md); en OpenCode, agente primario |
| 11 | `bin/install-agents` | Con el `sed`+`awk`, OpenCode recibía `tools` como lista de nombres de Claude Code y claves que no entiende (`maxTurns`, `effort`) | conversor que genera el frontmatter de OpenCode: `mode`, `model`, `tools: {write, edit, bash}` (validado con YAML) |
| 12 | `agents/*.md` | `claude-opus-4-6` / `claude-sonnet-4-6` obsoletos | Claude Code: alias `opus`/`sonnet` (no caducan); OpenCode: línea `# opencode-model:` |

## Mejoras e inconsistencias

- **Coordinación multi-agente:** plantilla `templates/TICKETS.md` (copiada por `new-project`). Reglas de propiedad, rama y revisor en orchestrator, builder y en las instrucciones del proyecto.
- **`templates/AUDIT_PROMPTS.md`:** nueva sección *Pre-seeded findings* para proyectos existentes.
- **`CLAUDE_PROJECT_INSTRUCTIONS.md`:** nuevos elementos:
  - tipo de conversación "proyecto existente / auditoría";
  - reglas multi-agente;
  - campo `Tool/Branch/Tickets` en el session task block;
  - workstation "impact";
  - `includeIf`.
- **Ruta de las skills unificada** a `skills/<name>.md` en SETUP, README y `new-project`. Antes SETUP decía `skills/<name>/SKILL.md`.
- **Seguridad de la API key:**
  - SETUP y README ya no recomiendan hardcodear la clave en `.zshrc`; se usa `~/.secrets` con `chmod 600`.
  - Se aclara que la clave solo se necesita con `--api`.
- **Otras correcciones:**
  - `quality-auditor` y `audit-quality` usan `pip-licenses` en lugar del frágil `pip show | paste`.
  - La skill `ml-integration` lee el modelo de `LLM_MODEL` en lugar de `claude-sonnet-4-20250514`.

## Pendiente de verificar en tu máquina

- **Directorio de agentes de OpenCode.** El script usa `~/.config/opencode/agents/`. Confirma con la documentación de tu versión (context7) si es `agent/` o `agents/`.
- **Nombres de los modelos en OpenCode.** Ejecuta `opencode models anthropic` y ajusta las líneas `# opencode-model:` si los nombres difieren.
- **Recomendación: pon `~/.dev-infra` bajo git** (repo privado en `github-personal`). Así cada cambio al template es revisable y reversible.
