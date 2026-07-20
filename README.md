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

### Bots preditivos

Anteriormente, os bots e o Titã miravam somente na posição atual do adversário. Quando o jogador se movimentava lateralmente ou na diagonal, os disparos eram direcionados para o local em que ele estava, facilitando as esquivas.

Para melhorar esse comportamento, foi implementada uma mira preditiva que registra a velocidade e a direção recente do alvo. A velocidade é calculada comparando a posição atual com a posição do quadro anterior e passa por uma suavização para evitar mudanças bruscas na mira.

Com esses dados, o bot calcula uma possível posição futura do adversário considerando:

- A distância entre o atirador e o alvo;
- A velocidade e a direção do alvo;
- A velocidade do projétil;
- O nível de dificuldade.

A precisão da previsão varia entre 30% e 105%. Nos níveis menores, os bots antecipam apenas parcialmente o movimento, enquanto nos níveis maiores a previsão se aproxima de uma interceptação completa. A posição prevista também é limitada à área válida do mapa. Caso o ponto calculado esteja atrás de uma parede, o bot tenta utilizar uma previsão parcial. Se essa posição também estiver bloqueada, ele volta a mirar na posição atual do alvo. A mira preditiva foi aplicada aos disparos dos bots comuns e aos diferentes ataques do Titã, incluindo rajadas, tiros rápidos e projéteis de bazuca.

Com essa melhoria, os inimigos passaram a antecipar movimentos laterais e diagonais, tornando os combates mais desafiadores sem impedir que o jogador consiga desviar por meio de mudanças repentinas de direção.

### Correção de travamento nos limites do mapa

Os bots e o Titã apresentavam travamentos próximos aos cantos, paredes e limites da arena. Quando o trajeto direto estava bloqueado nos eixos horizontal e vertical, eles podiam insistir na mesma direção ou alternar continuamente entre movimentos opostos.

Para corrigir esse problema, foi implementado um sistema de pathfinding utilizando o algoritmo A*. A arena foi dividida em células de 30 pixels, classificadas como livres ou bloqueadas conforme a posição das paredes e dos limites do mapa. O cálculo da rota considera o tamanho de cada personagem. Bots comuns e o Titã utilizam grades de navegação diferentes, evitando que o Titã tente atravessar espaços adequados somente para personagens menores. O A* permite movimentação em oito direções e impede cortes diagonais por quinas. Depois que a rota é encontrada, os pontos desnecessários são removidos para tornar o deslocamento mais fluido.

Também foi implementado um ajuste para destinos inacessíveis. Quando o alvo está encostado em um canto que não pode ser ocupado por uma entidade maior, o destino é substituído pelo ponto livre mais próximo. Assim, o inimigo não insiste indefinidamente em uma coordenada impossível. A movimentação nos limites também foi modificada. Quando somente uma parte do movimento sairia da arena, essa componente é removida e a entidade continua se deslocando pela borda. Isso evita oscilações entre esquerda e direita ou entre cima e baixo.

Por fim, um contador identifica situações em que o personagem tentou se mover, mas não apresentou progresso. Quando um travamento verdadeiro é detectado, a rota atual é descartada e um novo caminho é calculado. Paradas normais nos limites não são consideradas travamentos.

Com essa melhoria, bots e Titã passaram a contornar obstáculos, respeitar os limites físicos do mapa e utilizar rotas compatíveis com seu tamanho, reduzindo comportamentos de rotação, oscilação e movimento sem progresso.

### Memória e perseguição do Titã

Durante a análise do comportamento do Titã, observou-se uma limitação significativa em sua inteligência artificial relacionada à perseguição dos adversários. Quando o jogador se desloca para trás de uma parede ou outro obstáculo, interrompendo a linha de visão direta, o Titã abandona imediatamente a perseguição e permanece inativo até que um novo alvo entre novamente em seu campo de visão. Esse comportamento demonstra que o agente não possui capacidade de manter informações sobre a última posição conhecida do inimigo nem de planejar ações para continuar a busca. Como consequência, o jogador consegue escapar facilmente apenas utilizando obstáculos do cenário, tornando o comportamento do inimigo previsível e reduzindo o nível de desafio e realismo da partida.
