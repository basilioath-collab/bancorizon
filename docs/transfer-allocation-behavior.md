# Comportamento esperado para transferência de demandas

## Problema observado

Na demanda `ENCT0093 - Validação Legado`, a transferência de atuação removeu a demanda da alocação do usuário de origem no momento da ação, mas edições posteriores feitas por outro usuário reintroduziram a alocação antiga no card. Esse comportamento quebra a expectativa operacional: depois que uma atuação é transferida, o usuário de origem não deve voltar a aparecer como alocado apenas porque alguém editou dados cadastrais, observações, status ou apontamentos da demanda.

## Comportamento que deve mudar no sistema

### 1. Transferência deve ser tratada como mudança de titularidade da alocação

Ao transferir uma demanda de um recurso para outro, o sistema deve encerrar a alocação ativa do recurso de origem e criar ou atualizar a alocação ativa do recurso de destino. A partir desse momento, o recurso de origem só deve permanecer visível no histórico, nos apontamentos já lançados ou em uma alocação encerrada com data final anterior à transferência.

### 2. Edição comum da demanda não deve reconstruir alocações antigas

Operações como alterar observações, status, prioridade, datas, prédio, focal, etapas documentais ou registrar apontamento não devem reaplicar uma versão antiga da lista de `allocations`. Essas telas devem enviar e persistir somente os campos editados, ou então mesclar a edição com o estado atual do banco antes de salvar.

### 3. A lista ativa de alocações deve ter uma fonte de verdade

O sistema precisa definir uma única fonte de verdade para saber quem está alocado no card da demanda. A recomendação é considerar `allocations` como o estado ativo atual e `transfer_history` como trilha de auditoria. Durante consolidação, importação ou rebuild por eventos, uma transferência mais recente deve prevalecer sobre snapshots antigos que ainda contenham o recurso de origem.

### 4. Transferências precisam ser idempotentes e protegidas contra sobrescrita

Se uma transferência já foi aplicada, reprocessar eventos ou salvar uma edição posterior não pode duplicar o destino nem restaurar a origem. O merge deve comparar versão, timestamp e histórico de transferência antes de aceitar uma lista de alocações enviada pelo cliente.

### 5. O card deve diferenciar alocação ativa de histórico

Para melhorar a acessibilidade operacional do banco, o card deve exibir apenas recursos atualmente alocados na área principal. Pessoas que já atuaram na demanda devem aparecer em uma seção separada, como "Histórico de atuação" ou "Transferências", com data, origem, destino e justificativa.

## Regra prática recomendada

Sempre que uma demanda tiver `transfer_history`, o sistema deve aplicar a seguinte regra antes de exibir ou salvar o card:

1. Ordenar transferências por `timestamp`.
2. Para cada transferência, remover da alocação ativa o `fromResourceId`, exceto se houver uma alocação ativa criada explicitamente depois da transferência.
3. Criar ou manter o `toResourceId` como alocação ativa com as horas/percentual transferidos.
4. Preservar o recurso de origem apenas em histórico, apontamentos e relatórios retroativos.

## Critérios de aceite

- Após transferir uma demanda, o usuário de origem deixa de aparecer como alocado no card principal.
- Alterações posteriores feitas pelo destino ou por outro usuário não restauram a alocação do usuário de origem.
- Rebuild, consolidação ou importação do snapshot não reverte transferências já aplicadas.
- O histórico de transferências continua preservado para auditoria.
- Apontamentos antigos do usuário de origem continuam visíveis como histórico, sem significar alocação ativa.

## Aplicação ao caso ENCT0093

No caso da `ENCT0093 - Validação Legado`, Arthur.Costa deve permanecer no histórico de apontamento e no `transfer_history`, mas não deve voltar para `allocations` depois da transferência para Carlos.Lourenco. O card ativo da demanda deve mostrar somente os recursos ainda responsáveis pela execução atual.
