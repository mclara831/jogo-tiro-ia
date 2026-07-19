# Jogo de Tiro - IA

## Melhorias implementadas

### Ranqueamento de ameaça e seleção de alvo do Titã

O Titã agora identifica a origem de todo dano efetivo que recebe. Cada acerto feito pelo jogador ou por um bot aliado gera **pontos de ameaça** para o respectivo atacante. Assim, o agente mantém um estado interno no formato `atacante -> { pontos, danoTotal, acertos, ultimoAcerto }` durante a batalha.

A seleção do alvo é tratada como uma busca por pontos de coleta sob a perspectiva do Titã: entre os oponentes vivos dentro do alcance de priorização, ele ordena os candidatos pelo total de pontos de ameaça e escolhe o primeiro do ranking. A distância e a recência do último acerto são usadas somente para desempatar candidatos com a mesma pontuação; elas não superam a prioridade dos pontos. Dessa forma, o Titã busca eliminar os oponentes na ordem de impacto que causam no combate.

O ranking é reavaliado a cada 4 segundos. Enquanto o alvo ainda for válido e estiver no alcance, a decisão é mantida, evitando mudanças de foco a cada quadro e permitindo uma janela de reação para a equipe. Caso o alvo seja derrotado ou saia do alcance, uma nova busca é feita imediatamente entre os candidatos válidos.

Há uma exceção de **finalização** acima do ranking: um oponente visível, dentro do alcance e com no máximo 10% de HP recebe foco imediato. Ao entrar nesse estado, o Titã cria um plano de execução com dois disparos preditivos: cada um recebe metade arredondada para cima da vida efetiva atual do alvo (HP + escudo). Portanto, dois acertos são suficientes para eliminar o alvo mesmo se ele possuir escudo. Se um projétil errar, a conclusão continua dependente de o Titã acertar os dois disparos planejados.

Após a escolha, o Titã continua usando:

- **A\*** para alcançar a posição do alvo prioritário quando uma parede bloqueia o trajeto;
- **Memória e pontos de investigação** quando o alvo prioritário deixa de estar visível;
- **Mira preditiva**, que estima a posição futura do alvo usando sua velocidade suavizada e a velocidade do projétil.

O painel do modo Chefão exibe a prioridade atual e sua pontuação, tornando o comportamento observável durante os testes.

### Memória e perseguição do Titã

Durante a análise do comportamento do Titã, observou-se uma limitação significativa em sua inteligência artificial relacionada à perseguição dos adversários. Quando o jogador se desloca para trás de uma parede ou outro obstáculo, interrompendo a linha de visão direta, o Titã abandona imediatamente a perseguição e permanece inativo até que um novo alvo entre novamente em seu campo de visão. Esse comportamento demonstra que o agente não possui capacidade de manter informações sobre a última posição conhecida do inimigo nem de planejar ações para continuar a busca. Como consequência, o jogador consegue escapar facilmente apenas utilizando obstáculos do cenário, tornando o comportamento do inimigo previsível e reduzindo o nível de desafio e realismo da partida.



## Problemas:

Quando o boss vẽ os adversários, mas está em um local que não consegue passar, ele continua tentando atirar nos adversários, em vez de procurar um caminho melhor para chegar até o adversário e atacá-lo.
