# NUMBER-GUESSER-EXTREME

Projeto para a disciplina de Circuitos Digitais da UFCA (2026.1)


# RELATORIO

Abaixo seguem descrições de todos os componentes lógicos do projeto.

# Circuitos de Auxilio

### HOLD

O circuito Hold, tendo como entrada um botão, tem como saída 1 somente no ciclo em que o botão é pressionado, ou seja, mesmo que o botão continue a ser pressionado, depois de um ciclo a saída volta a ser 0, e só é 1 novamente quando o botão é solto e pressionado novamente.
O circuito tem como fundamentos principais dois registradores de 1 bit, `X `e `Y`.
`X=1` quando o botão está pronto para ser pressionado novamente, ele é ativado primariamente quando a entrada é zero.
`Y` existe para permitir que o primeiro pressionamento ative caso a entrada inicia em 1, é apenas uma medida de segurança para casos raros.
Basicamente, o circuito segue a expressão `(X==1 && B==1) || (Y==0 && B==1)`, onde `B` é a entrada.

### WAIT

O circuito Wait, tendo como entrada um bit, faz com que a saída seja essa entrada somente um ciclo depois.
Exemplo, onde `A` é a entrada e `B` é a saída.

`A=1, B=0` (ciclo) <br>
`A=1, B=1` (ciclo) <br>
`A=0, B=1` (ciclo) <br>
`A=0, B=0`

Esse circuito tem como base três registradores de 1 bit, `X`, `Y` e `Select`. Todos começam com valor 0. A cada ciclo, `Select` troca seu bit para o oposto, e a saída é `X` quando `Select=0` e `Y` quando `Select=1`.
A cada ciclo, o valor `X` ou `Y` é alterado para a entrada, sendo que o valor alterado é o oposto ao que está sendo utilizado como saída (ou seja, se `X` é a saída, o valor alterado é `Y`, e vice-versa).

Assim, quando `X` é a saída, `Y` será a proxima saída, e está recebendo o valor da entrada. No próximo ciclo, `Y` é a saída e possui o valor da entrada anterior, enquanto `X` está agora recebendo o valor da entrada atual. E assim sucessivamente.

### SLOWCLOCK

O circuito SlowClock basicamente funciona como um clock que só muda de valor a cada 32 ciclos do seu clock de entrada. Usa-se um contador e um comparador, incrementando esse contador a cada ciclo do clock de entrada, e quando o valor do contador é 0, ou seja, ocorreu overflow sobre seu valor máximo, a saída passa a ser 1.
O contador interno possui 8 bits apenas para fácil modificação, a frequencia do clock é modificada apartir do valor máximo do contador.

### SOMADOR

O somador de 4 para 5 bits é simplesmente um somador do logisim onde o c-out é utilizado como bit mais significante da saída, e os outros bits são o resultado da soma.

### DIFERENÇA

A diferença permite achar o modulo da subtração das entradas `X` e `Y`,
isso é feito através do uso de dois subtratores (`X-Y` e `Y-X`) e um multiplexador, o multiplexador escolhe o valor positive entre as subtrações, obtendo essa informação apartir do b-out de `X-Y`
(pois se o b-out é 1, `X-Y < 0`, o que significa por sua vez que `Y-X > 0`).

# CFG

Circuitos usados para configurar o cronometro.

### CFGBUTTONS

O circuito CfgButtons simplesmente filtra todos os botões relevantes a configuração do cronometro através de Holds, ou seja, nenhum dos botões pode ser simplesmente segurado. Isso ajuda a manter o controle dos botões em frequencias mais altas.
É o seu próprio circuito apenas por questão de organização.

### CFGVALUE

O CfgValue armazena um dos valores configurados e possui a lógica de incrementação desse valor quando o botão relevante é pressionado,
assim como também a lógica de manter o valor no seu limite (e.g. dezenas de minutos só podem ir de 0 a 5, unidades de 0 a 9)

### CFG

O circuito principal de configução do cronometro, ele armazena os valores das unidades e dezenas de segundos e minutos individualmente, mas sua saída é um valor de 16 bits que é o total em segundos. O circuito também armazena se o modo de configuração está ativado ou não, tendo também este valor como saída.

# GUESS

Circuitos usados em relação aos palpites dos jogadores.

### PLAYERGUESS

Armazena e permite a manipulação do palpite de um jogador. Apenas o jogador que tem a vez pode causar mudanças na memoria deste circuito, logo tem como entrada o jogador atual e uma constante que indica a qual jogador esse circuito "pertence",
impedindo todos os botões de funcionarem caso os valores não sejam iguais (isso é feito através de portas AND). Outra entrada que barra o funcionamento do dos botões é a entrada inferior direita, que indica se o modo de teste de pontuação está ativado
(sendo este o modo onde os palpites são comparados aos valores secretos e as luzes de acerto podem estar acessas),
se este modo estiver ativado então apenas o botão avançar tem qualquer efeito.

Quando estas condições estiverem cumpridas, os botões relevantes podem ser pressionados. Os registradoes da direita são simplesmente os palpites, o registrador central indica qual componente da coordenada esta se fazendo o palpite sobre, os registradores inferiores indicam se as luzes de "jogando x" ou "jogando y" estão acessas (junta-se as suas saída a condição de que o jogador correspondente tem a vez e que o palpite atual está sendo feito sobre o componente correspondente ao registrador)

As saídas são os palpites, se as luzes de "jogando x" e "jogando y" estão acessas de fato, e se o modo de teste de pontuação foi ativado no ciclo anterior (ou seja, o botão de avançar foi pressionado depois dos dois palpites terem sido lançados).

### SCORETESTMODE

Tem como saída se o modo de teste de pontuação está ou não ativo, ele determina isso apartir da saída do PlayerGuess correspondente e se o botão avançar foi pressionado. Caso tenha sido, pressionar o botão de avançar novamente desativa o modo de teste de pontuação. A segunda saída será 1 quando o modo for desativado neste ciclo (modo ativado e botão de avançar pressionado ao mesmo tempo)

### RESETCONDITION

Condição de quando a coordenada secreta dever ter um novo valor atribuido a ela. As entradas são se os palpites estão corretos, se o botão reiniciar foi pressionado, e se o modo de teste de pontuação acabou de ser desativo.
Caso os palpites estejam certos e o modo desativado, ou quando o botão de reiniciar for pressionado, a saída será 1.

### GUESSLOGIC

O circuito GuessLogic armazena a coordenada secreta e tem como função compara-la com os palpites de um jogador, tendo como saída se os palpites estão certos, ou se a soma está correta, ou quão proxima a soma está da soma dos componentes da coordenada, na forma de dois valores de cor.
A coordenada secreta se reinicia apartir do valor de ResetCondition, e as cores são baseadas na Diferença (junto a left shift). O resto é um uso básico de somadores e comparadores.

# CRONOMETRO

Circuitos relacionados ao cronometro.

### PLAYERTIME

Armazena o tempo atual de um jogador e decrementa a cada ciclo de um SlowClock o valor em 1, mantendo-se em 0 quando chega a esse valor.

Também é função desse circuito dividir os segundos de volta a unidades e dezenas de segundos e minutos para serem mostrados na interface de usuário, fazendo isso apartir de divisões com resto.

### CHANGEPLAYER

Condição de quando o jogador deve passar o turno ao próximo. Checa se o tempo do jogador correspondente é 0 ou se ele avançou seu turno manualmente.

### PLAYERCRONOMETERGATEA

Condição para que o jogador A tenha seu cronometro modificado. Tem que ser o turno do jogador A ou o modo CFG tem que estar ligado, e o clock deve estar em 1.

### PLAYERCRONOMETERGATEB

Condição para que o jogador B tenha seu cronometro modificado. Tem que ser o turno do jogador B ou o modo CFG tem que estar ligado, e o clock deve estar em 1.

### CRONOMETRO

Circuito principal para manipulação do tempo dos jogadores, e também de qual jogador é a vez. Possui como memoria o jogador atual, que é modificado sempre que ChangePlayer tiver saída 1.
Tem como entradas se o turno foi avançado, se o modo de cfg está desativado, se o botão reset foi pressionado, e o tempo que foi configurado em segundos de ambos os jogadores.

# SCORE

Circuitos relacionados à pontuação dos jogadores.

### SCORE

Armazena a pontuação de um jogador e também possui a lógica de incrementação da pontuação. Tem como entradas se o botão de avançar do jogador correspondente foi pressionado, se os palpites estão corretos, e se o botão reset foi pressionado.
Quando o botão reset for pressionado, o valor retorna a zero, caso as outras condições sejam cumpridas, a pontuação aumenta em 1.

### WINCONDITION

Condição de vitoria de um jogador. Se ele tiver pontuação maior que seu competidor e ambos os tempos estejam esgotados, ou se ele acertou 15 vezes, ele ganha o jogo e a saída é 1.

# OUTROS

### ENDGATE (1 bit, 4 bits, 8 bits)

Se o jogo estiver terminado, torna a saída 0. Usado para impedir que as luzes se mantenham acesas após o termino do jogo.

# MAIN

Componente principal lógico do projeto. Tem como entrada os botões da interface de usuario e como saída os valores a serem mostrados.





