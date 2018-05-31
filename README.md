# Emanuel-Mendes

# CONTROLE DE MOTOR BRUSHED POR PWM


### 1.DESCRIÇÃO
Controle de velocidade de motor DC Brushed utilizando a modulação PWM, a partir de um sinal de entrada, propiciando a inversão e o controle do giro do motor. O objetivo é controlar um motor de maneira mais eficiente, sem perda do torque, de forma a facilitar o ajuste de velocidade por software. O projeto deve incluir sinais de saída PWM em dois dos pinos do hardware a fim de possibilitar a inversão no sentido do giro ao habilitar/desabilitar um dos PWM’s por vez, sendo este controle determinado por um outro pulso de entrada (PWM) em uma porta da placa. Observe o diagrama de blocos abaixo:
![screenshot 1](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/DiagramaBlocos.png)

### 2.HARDWARES EM USO

Utilizou-se a placa STM32F407G-DISC1 (Discovery) , córtex M4, além de uma placa auxiliar para geração de PWM na entrada (entenda entrada como uma porta da Discovery), bem como um driver projetado para controle de motor Brushed, topologia Bootstrap. Para o bom funcionamento do projeto, é necessário também uma fonte de tensão 15V estável com capacidade de corrente de cerca de 2A . Observe o esquemático do Driver:
![screenshot 2](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/Driver.pdf)


### 3. FUNCIONAMENO

Será a Discovery excitada por uma fonte de PWM externa com duty-cycle variável. Para a saída, espera-se obter em um dos pinos a serem definidos no decorrer do projeto, e não simultaneamente em ambas as portas de saída, um PWM cujo duty-cycle varia de 0 a 100% - aproximadamente. Dessa forma, aciona-se o Driver MorpheusV5 - utilizada no projeto Batalha de Robôs 2017 – de acordo com o diagrama temporal abaixo: 
![screenshot 3](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/DiagramaTempo.PNG)

Observe que desse diagrama depreende-se que para girar o motor em um sentido, aplicamos um sinal PWM no pino IN_1. Para girar no sentido contrário, aplica-se sinal PWM no pino IN_2. Em ambos os casos, o pino não utilizado deve encontrar-se em nível baixo. 

### 4.PINAGEM

Em construção

### 5.FLUXOGRAMA DO FIRMWARE

Em construção

### 6.REFERÊNCIAS

 [DATASHEET IR2184 - DHT11]https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/ir2184.pdf)

[MANUAL DE REFERÊNCIA - STM32F4 Discovery](http://www.st.com/content/ccc/resource/technical/document/reference_manual/3d/6d/5a/66/b4/99/40/d4/DM00031020.pdf/files/DM00031020.pdf/jcr:content/translations/en.DM00031020.pdf)
