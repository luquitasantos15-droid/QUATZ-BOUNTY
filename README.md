# UltimateBounty

Sistema de recompensas por jogadores para **Paper 1.21.x (Java 21)**. Qualquer jogador coloca dinheiro na cabeça de outro;
quem eliminar o alvo em PvP recebe o total acumulado diretamente na conta do **EssentialsX Economy** (via **Vault**).

## Requisitos
- Paper 1.21.x, Java 21
- Vault + EssentialsX (Economy)
- O driver SQLite é baixado automaticamente pelo servidor na primeira inicialização (precisa de internet nessa vez).

## Compilação
Local (precisa de JDK 21 e Gradle 8.x): `gradle build` -> `build/libs/UltimateBounty-1.0.0.jar`.
No GitHub: o arquivo `.github/workflows/build.yml` compila a cada push; baixe o `.jar` em **Actions -> Artifacts**.

## Instalação
1. Coloque o `.jar` em `plugins/` junto com Vault e EssentialsX.
2. Reinicie o servidor. São criados `config.yml`, `messages.yml` e `bounties.db` (SQLite).
3. Ajuste a configuração e use `/bounty reload`.

## Comandos
| Comando | Descrição | Permissão |
|---|---|---|
| `/bounty` | Menu principal | `ultimatebounty.use` |
| `/bounty add <jogador> <valor>` | Coloca recompensa (aceita 5000, 10k, 2.5m, 1b) | `ultimatebounty.add` |
| `/bounty check <jogador>` | Consulta a recompensa | `ultimatebounty.check` |
| `/bounty top` | Maiores recompensas | `ultimatebounty.top` |
| `/bounty list [página]` | Recompensas ativas | `ultimatebounty.use` |
| `/bounty help` | Ajuda | `ultimatebounty.use` |
| `/bounty reload` | Recarrega config e mensagens | `ultimatebounty.reload` |

`ultimatebounty.admin` concede todas as permissões.

## Como funciona
- **Acumulativa:** cada contribuição é somada ao total do alvo e guardada com quem contribuiu.
- **Pagamento:** só quando um jogador é eliminado por outro jogador (suicídio, queda sem combate, mobs e morte natural não pagam).
  Com `allow-indirect-kills: false` só paga se o último dano foi direto (corpo a corpo ou projétil).
- **Anti-duplicação:** a recompensa sai da memória antes do pagamento; no banco, remoção + histórico + pagamento acontecem
  em uma transação SQLite (`payments.history_id` é UNIQUE). Se o depósito no Vault falhar, tudo é revertido e a recompensa continua ativa.
- **Anti-farming** (`prevent-farming`): mesmo IP entre assassino e alvo, quem contribuiu não recebe, e cooldown por par
  assassino/alvo (`farming-cooldown-minutes`). Em servidor de teste local, desligue `protection.block-same-ip`.
- **Banco:** tabelas `bounties`, `contributions`, `bounty_history`, `payments`. Tudo persiste entre reinícios.

## Estrutura
`UltimateBounty` (main), `BountyCommand`, `BountyManager`, `Bounty`, `BountyListener`, `EconomyManager`,
`DatabaseManager`, `GUI`, `GuiListener`, `InputManager`, `TopBounty`, `ProtectionManager`, `Messages`, `Utils`.
