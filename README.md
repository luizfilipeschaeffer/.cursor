# .cursor

> Configuração de Agentes, Skills e Workflows para [Cursor](https://cursor.com).

**Repositório:** [github.com/luizfilipeschaeffer/.cursor](https://github.com/luizfilipeschaeffer/.cursor)

---

## Estrutura do repositório

| Pasta / arquivo   | Descrição                                      |
| ----------------- | ---------------------------------------------- |
| **agents/**       | Agentes especialistas (personas por domínio)   |
| **commands/**     | Comandos de barra (workflows)                  |
| **plans/**        | Planos e tarefas (.plan.md)                    |
| **rules/**        | Regras globais (ex.: `default.mdc`)            |
| **scripts/**      | Scripts de validação e checklist               |
| **skills/**       | Módulos de conhecimento por domínio            |
| **ui-ux-pro-max/**| Estilos, paletas e dados para UI/UX            |
| **ARCHITECTURE.md** | Visão geral da arquitetura e componentes     |
| **TASKS.md**      | Tarefas e pendências                           |
| **mcp.json**      | Configuração de servidores MCP                 |

---

## O que está incluído

| Componente | Quantidade | Descrição                                           |
| ---------- | ---------- | --------------------------------------------------- |
| **Agents** | 19         | Personas por domínio (frontend, backend, security, QA, etc.) |
| **Skills** | 45+        | Módulos de conhecimento (frontend, API, DB, mobile, etc.)   |
| **Commands** | 11       | Procedimentos por comando de barra                   |

---

## Uso

### Usar em um projeto

Clone ou use este repositório como referência e copie a pasta `.cursor` (ou o conteúdo) para a raiz do seu projeto. No Cursor, as regras em `rules/`, os agentes em `agents/` e os skills em `skills/` passam a ser usados automaticamente conforme o contexto.

### Agentes

O roteamento de agentes é automático: o Cursor escolhe o especialista conforme o pedido.

- **Exemplos:** "Adicionar autenticação JWT" → `@security-auditor` + `@backend-specialist`  
- "Corrigir botão do dark mode" → `@frontend-specialist`  
- "Login retorna 500" → `@debugger`

### Comandos de barra

| Comando          | Descrição                          |
| ---------------- | ---------------------------------- |
| `/brainstorm`    | Explorar opções antes de implementar |
| `/create`        | Criar features ou apps             |
| `/debug`         | Debug sistemático                  |
| `/deploy`        | Deploy da aplicação                |
| `/enhance`       | Melhorar código existente          |
| `/orchestrate`   | Coordenação multi-agente           |
| `/plan`          | Quebra de tarefas                  |
| `/preview`       | Preview local das mudanças         |
| `/status`        | Status do projeto                  |
| `/test`          | Gerar e rodar testes               |
| `/ui-ux-pro-max` | Design com estilos e paletas       |

### Skills

Os skills são carregados conforme o contexto da tarefa. O Cursor usa as descrições em cada `SKILL.md` e aplica o conhecimento correspondente.

---

## Documentação

- **[ARCHITECTURE.md](ARCHITECTURE.md)** – Visão da arquitetura, agentes e skills.
- **[TASKS.md](TASKS.md)** – Tarefas e pendências do repositório.

---

## Licença

MIT.
