# Emanuel-Mendes

# CONTROLE DE MOTOR BRUSHED POR PWM


### 1.DESCRIÇÃO
Controle de velocidade de motor DC Brushed utilizando a modulação PWM, a partir de um sinal de entrada, propiciando a inversão e o controle do giro do motor. O objetivo é controlar um motor de maneira mais eficiente, sem perda do torque, de forma a facilitar o ajuste de velocidade por software. O projeto deve incluir sinais de saída PWM em dois dos pinos do hardware a fim de possibilitar a inversão no sentido do giro ao habilitar/desabilitar um dos PWM’s por vez, sendo este controle determinado por um outro pulso de entrada (PWM) em uma porta da placa. Observe o diagrama de blocos abaixo:
![Figura 1](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/DiagramaBlocos.png)

### 2.HARDWARES EM USO

Utilizou-se a placa STM32F407G-DISC1 (Discovery) , córtex M4, além de uma placa auxiliar para geração de PWM na entrada (entenda entrada como uma porta da Discovery), bem como um driver projetado para controle de motor Brushed, topologia Bootstrap. Para o bom funcionamento do projeto, é necessário também uma fonte de tensão 15V estável com capacidade de corrente de cerca de 2A . Observe o esquemático do Driver:
![Figura 2](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/Driver.PNG)


### 3. FUNCIONAMENO

Será a Discovery excitada por uma fonte de PWM externa com duty-cycle variável. Para a saída, espera-se obter em um dos pinos a serem definidos no decorrer do projeto, e não simultaneamente em ambas as portas de saída, um PWM cujo duty-cycle varia de 0 a 100% - aproximadamente. Dessa forma, aciona-se o Driver MorpheusV5 - utilizada no projeto Batalha de Robôs 2017 – de acordo com o diagrama temporal abaixo: 
Observe que desse diagrama depreende-se que para girar o motor em um sentido, aplicamos um sinal PWM no pino IN_1. Para girar no sentido contrário, aplica-se sinal PWM no pino IN_2. Em ambos os casos, o pino não utilizado deve encontrar-se em nível baixo.
![Figura 3](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/DiagramaTempo.PNG) 

### 4.PINAGEM
Observe a pinagem utilizada abaixo:

![Figura 5](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/pinos.PNG)

Abaixo o código:

```
#include "stm32f4xx.h"
#include "stm32f4xx_exti.h"
#include "stm32f4xx_syscfg.h"
#include "misc.h"
#include "stm32f4_discovery.h"


int main(void)
{
  int i = 0;

  /* Initialize LEDs */
  STM_EVAL_LEDInit(LED3);
  STM_EVAL_LEDInit(LED4);
  STM_EVAL_LEDInit(LED5);
  STM_EVAL_LEDInit(LED6);

  /* Turn on LEDs */
  STM_EVAL_LEDOn(LED3);
  STM_EVAL_LEDOn(LED4);
  STM_EVAL_LEDOn(LED5);
  STM_EVAL_LEDOn(LED6);

  //INICIALIZANDO CLOCKS
  RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE); //HABILITOU O CLOCK NA PORTA
  RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE); //HABILITOU O CLOCK DO TIMER2
  RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE); //HABILITOU O CLOCK DO TIMER1
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_SYSCFG, ENABLE); //HABILITOU O CLOCK DO SYSCFG

  //INICIALIZA O GPIO DE SAIDA
  GPIO_InitTypeDef GPIO;
  GPIO.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7; // SETA PINOS 6 E 7 E RECEBEM AMBOS AS CONFIGURAÇÕES DA ESTRUTURA
  GPIO.GPIO_Mode = GPIO_Mode_AF; //FUNCAO ALTERNATIVA PRA ESSE GPIO
  GPIO.GPIO_OType = GPIO_OType_PP; //(OUTPUT TYPE) PINOS EM PUSH-PULL
  GPIO.GPIO_PuPd = GPIO_PuPd_NOPULL; //SEM PULL-UP OU PULL-DOWN
  GPIO.GPIO_Speed = GPIO_Speed_25MHz; //FREQUENCIA MAXIMA PARA O PWM
  GPIO_Init(GPIOA, &GPIO); // ESSA FUNCAO INICIALIZA O GPIO COM A ESTRUTURA DEFINIDA ATÉ A LINHA PASSADA
                           // O GPIOA ESTÁ CONFIGURADA DE ACORDO COM A ESTRUTURA DE NOME "GPIO"

  //INICIALIZA O GPIO DE ENTRADA
  GPIO.GPIO_Pin = GPIO_Pin_0; // SETA PINO 0 PARA RECEBER AS CONFIGURAÇÕES DA ESTRUTURA
  GPIO.GPIO_Mode = GPIO_Mode_IN; //FUNCAO DE ENTRADA PRA ESSE PINO
  GPIO.GPIO_PuPd = GPIO_PuPd_NOPULL; //SEM PULL-UP OU PULL-DOWN
  // NAO USAMOS NA ENTRADA O _OType E _Speed PARA ENTRADAS
  GPIO_Init(GPIOA, &GPIO); //GPIOA É INICIALIZADO COMO DESCRITO NESSA ESTRUTURA ATÉ AGORA.

  //CONFIGURANDO FUNCAO ALTERNATIVA DAS PORTAS
  GPIO_PinAFConfig(GPIOA,GPIO_PinSource6,GPIO_AF_TIM3); //CONF A PORTA PARA SERVIR AO TIMER3
  GPIO_PinAFConfig(GPIOA,GPIO_PinSource7,GPIO_AF_TIM3); //CONF A PORTA PARA SERVIR AO TIMER3

  //CONFIGURAR O CONTADOR DO TIMER3
  TIM_TimeBaseInitTypeDef TIM; //ESTRUTURA TIM
  TIM.TIM_Prescaler = 14-1; //DIVIDE O CLOCK POR 14 ,OU SEJA, CLK'= 84MHz/14 = 6MHz
  TIM.TIM_Period = 300-1; //CONTA DE 0 A 299 ATÉ DAR O OVERFLOW DO TIMER1
                          //DIVIDE 6MHz POR 300, DE FORMA A DAR UM CLOCK DE 20KHz
  TIM.TIM_CounterMode = TIM_CounterMode_Up; //CONTADOR CRESCENTE.
  TIM_TimeBaseInit(TIM3, &TIM); //TIM1 É INICIALIZADO COMO DESCRITO NESSA ESTRUTURA ATÉ AGORA.

  //CONFIGURAR CONTADOR DO TIMER 2 (UTILIZANDO A ESTRUTURA ACIMA)
  TIM.TIM_Prescaler = 42-1; //DIVIDE O CLOCK POR 42 ,OU SEJA, CLK'= 84MHz/42 = 2MHz (VALOR ARBITRADO DE 0.5us PARA LER A ENTRADA)
  TIM.TIM_Period = 40000-1; //CONTA DE 0 A 39999 ATÉ DAR O OVERFLOW DO TIMER2
                            //DIVIDE 2MHz POR 400000, DE FORMA A DAR UM CLOCK DE 5Hz
  TIM.TIM_CounterMode = TIM_CounterMode_Up; //CONTADOR CRESCENTE (NAO PRECISA ESTAR AQUI, VISTO QUE JÁ FOI SETADO ACIMA)
  TIM_TimeBaseInit(TIM2, &TIM); //TIM2 É INICIALIZADO COMO DESCRITO NESSA ESTRUTURA ATÉ AGORA.

  //DEFINIR PWM SAIDA
  TIM_OCInitTypeDef OC;
  OC.TIM_OCMode = TIM_OCMode_PWM1; //SETA O TIPO DE PWM
  OC.TIM_OCPolarity = TIM_OCPolarity_High; // O NIVEL INATIVO AQUI É O 'O'
  OC.TIM_OutputState = TIM_OutputState_Enable;
  OC.TIM_Pulse = 180;
  TIM_OC1Init(TIM3, &OC); //CONFIGURA O CANAL 1 DO PWM
  TIM_OC2Init(TIM3, &OC); //CONFIGURA O CANAL 2 DO PWM

  //HABILITA OS TIMERS
  TIM_Cmd(TIM3, ENABLE); //HABILITA O TIMER1
  TIM_Cmd(TIM2, ENABLE); //HABILITA O TIMER2

  SYSCFG_EXTILineConfig(EXTI_PortSourceGPIOA, EXTI_PinSource0); //GPIOA COMO FONTE DO INTERRUPT 0

  EXTI_InitTypeDef EXTIs;
  EXTIs.EXTI_Line = EXTI_Line0;	//DEFINE O USO PARA O LINE0
  EXTIs.EXTI_LineCmd = ENABLE; //HABILITA O LINECMD
  EXTIs.EXTI_Mode = EXTI_Mode_Interrupt; //SETA O MODO INTERRUPT PARA A LINHA
  EXTIs.EXTI_Trigger = EXTI_Trigger_Rising_Falling; //GERA INTERRUPT NA SUBIDA E NA DESCIDA
  EXTI_Init(&EXTIs); //EXTI É INICIALIZADO COMO DESCRITO NESSA ESTRUTURA ATÉ AGORA.

  NVIC_InitTypeDef NVICs;
  NVICs.NVIC_IRQChannel = EXTI0_IRQn; //CONFIGURA QUE SERÁ USADO O INTERRUPT0
  NVICs.NVIC_IRQChannelPreemptionPriority = 0; //DEFINE A PRIORIDADE DO INTERRUPT
  NVICs.NVIC_IRQChannelSubPriority = 0; //IDEM
  NVICs.NVIC_IRQChannelCmd = ENABLE; //"LIGA" O INTERRUPT
  NVIC_Init(&NVICs); //NVIC É INICIALIZADO COMO DESCRITO NESSA ESTRUTURA ATÉ AGORA.

  // LOOP PRINCIPAL
    while (1)
    {
        while (!GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_0)) __WFI(); // DETECTA RISING EDGE
        int32_t t1 = TIM_GetCounter(TIM2); // CAPTURA CONTADOR DO TIM2 ATE OCORRER A BORDA DE SUBIDA

        while (GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_0)) __WFI(); // DETECTA FALLING EDGE
        int32_t t2 = TIM_GetCounter(TIM2); // CAPTURA CONTADOR DO TIM2 ATE OCORRER A BORDA DE DESCIDA

        // DUTY-CYCLE PWM
        int32_t dt = t2 - t1;
        if (dt < 0) dt += 40000; // CORREÇÃO DO PWM

        // SAÍDA DE PWM
        int32_t out = (dt - 3000)/3;

        // RESTRIÇÃO DE SAÍDA
        if (out < -290) out = -290;
        else if (out > 290) out = 290;

        // CONTROLE DOS CANAIS DE PWM
        if (out > 0)
        {
            TIM_SetCompare1(TIM3, out); //PWM NO CANAL POSITIVO
            TIM_SetCompare2(TIM3, 0); //PWM NO CANAL NEGATIVO
        }
        else
        {
            TIM_SetCompare1(TIM3, 0); //PWM NO CANAL POSITIVO
            TIM_SetCompare2(TIM3, -out); //PWM NO CANAL NEGATIVO
        }
    }
}

void EXTI0_IRQHandler(void) //CHAMADA QUANDO OCORRE A INTERUPT
{
    if (EXTI_GetITStatus(EXTI_Line0))
        EXTI_ClearITPendingBit(EXTI_Line0); //ZERA O BIT DO EXTI_LINE0 QUE INDICA A INTERRUPÇÃO
}

/*
 * Callback used by stm32f4_discovery_audio_codec.c.
 * Refer to stm32f4_discovery_audio_codec.h for more info.
 */
void EVAL_AUDIO_TransferComplete_CallBack(uint32_t pBuffer, uint32_t Size){
  /* TODO, implement your code here */
  return;
}

/*
 * Callback used by stm324xg_eval_audio_codec.c.
 * Refer to stm324xg_eval_audio_codec.h for more info.
 */
uint16_t EVAL_AUDIO_GetSampleCallBack(void){
  /* TODO, implement your code here */
  return -1;
}

```

### 5.FLUXOGRAMA DO FIRMWARE

Com base na programação do projeto, foi montado um fluxograma contendo as funcionalidades do código e a sequência na qual cada passo ocorre, levando em consideração todos os loops infinitos. Observe o diagrama:
![Figura 5](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/FLUXOGRAMA.png)

### 6.VIDEO DO FUNCIONAMENTO
[![video](https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/Capture.PNG)](https://www.youtube.com/watch?v=K7pEcNUHPrc&feature=youtu.be)

### 7.REFERÊNCIAS

 [DATASHEET IR2184 - DHT11]https://github.com/Microcontroladores2018/Emanuel-Mendes/blob/master/ir2184.pdf)

[MANUAL DE REFERÊNCIA - STM32F4 Discovery](http://www.st.com/content/ccc/resource/technical/document/reference_manual/3d/6d/5a/66/b4/99/40/d4/DM00031020.pdf/files/DM00031020.pdf/jcr:content/translations/en.DM00031020.pdf)
