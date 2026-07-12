# Jogo de Tiro - IA

## Melhoras implementas

### Memória e perseguição do Titã

Durante a análise do comportamento do Titã, observou-se uma limitação significativa em sua inteligência artificial relacionada à perseguição dos adversários. Quando o jogador se desloca para trás de uma parede ou outro obstáculo, interrompendo a linha de visão direta, o Titã abandona imediatamente a perseguição e permanece inativo até que um novo alvo entre novamente em seu campo de visão. Esse comportamento demonstra que o agente não possui capacidade de manter informações sobre a última posição conhecida do inimigo nem de planejar ações para continuar a busca. Como consequência, o jogador consegue escapar facilmente apenas utilizando obstáculos do cenário, tornando o comportamento do inimigo previsível e reduzindo o nível de desafio e realismo da partida.

A partir disso, foi implementada a busca utilizando o A* no último ponto visto o adversário, quando o boss não visualiza mais ninguém em seu ponto de visão.
Foi feita uma melhora em relação ao foco de ataque no titã: notou-se que em certos momentos o titã travava pois estava envonvidos por muitos adversários. Dessa maneira, foi feita uma atualização em que com base no adversário mais próximo, ele estabelece o foco de ataque de 3 segundos, passado esse tempo, ele procura o adversário mais próximo novamente para focar no ataque. Isso reduziu o travamento do titã.


## Problemas:

Quando o boss vẽ os adversários, mas está em um local que não consegue passar, ele continua tentando atirar nos adversários, em vez de procurar um caminho melhor para chegar até o adversário e atacá-lo.