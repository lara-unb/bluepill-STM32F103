# Ambiente de desenvolvimento para a placa genérica "blue pill" com microcontrolador STM32F103C8T6, para Windows

## Introdução

Nesse documento estão os procedimentos para preparar o ambiente de desenvolvimento para a placa genérica "blue pill" com o microcontrolador STM32F103C8T6. 

O STM32F103C8T6 é um microcontrolador de 32 bits poderoso, com controlador USB próprio. O ambiente de desenvolvimento do seu fabricante permite uma 
rápida configuração, além da possibilidade de incorporar o FreeRTOS.

Os procedimentos são para Windows. Será necessário baixar da internet arquivos de instalação, mas os links estão disponibilizados nesse guia.

Além do ambiente de desenvolvimento de software e da bluepill, é necessário adquirir uma gravadora STLink/V2. Há também clones de baixo custo no mercado.

## IDE

### SystemWorkbench for STM32

O SystemWorkbench for STM32 (SW4STM32) é uma ferramenta completa, vindo com IDE Eclipse, compilador GCC, OpenOCD para depuração além de drivers para a gravadora/depuradora STLINK.
Os arquivos e roteiro de instalação se encontram no link http://www.openstm32.org

Fontes:

http://www.openstm32.org/forumthread2405

http://www.openstm32.org


### Eclipse + GCC (pasta Eclipse+GCC+OpenOCD+FreeRTOS)

Existe também a possibilidade de se construir um ambietnte Eclipse com GCC, OpenOCD e FreeRTOS. Esse caminho é indidado para os mais experientes. 
Existe sites mostrando como proceder. Como o SW4STM32 se mostrou confiável e de fácil configuração, essa opção não é apresentada aqui.


## Gravadora/Depuradora STlink

Esse procedimento não é necessário se foi instadado o "SystemWorkbench for STM32".

Fontes:

https://onetransistor.blogspot.com.br/2017/11/stm32-bluepill-arduino-ide.html


## Biblioteca STM32CubeF1

STM32CubeF1 é uma biblioteca de drivers modulares composta de camada de baixo nível, camada HAL e camada de aplicação (incluindo RTOS). 

Para baixar essa biblioteca, use o link https://www.st.com/en/embedded-software/stm32cubef1.html

Para testar a instalação, pode-se compilar um dos exemplos com o SW4STM32. 

a) Inicie o SW4STM32.

b) Clique em "File->Switch Workspace->Other" e aponte para o diretório de trabalho definido na instalação (e.g., C:\AC6\workspace) ou qualquer outro  
workspace directory (e.g., C:\meu_projeto). O exemplo será importado para essa pasta, de forma a não se alterar a instalação original da STM32CubeF1

c) Clique em "File->Import", selecione "General->'Existing Projects into Workspace'" e clique "Next".

d) Com "Select root directory" selecionado, cloque em "Browse" e aponte para o diretorio "C:\STM32\STM32Cube_FW_F1_V1.6.0\Projects\STM32F103RB-Nucleo\Examples\GPIO\GPIO_IOToggle"

e) Em "Options" certifique-se que "Copy projects into workspace" não está selecionado. Selecione essa opção se desejar modificar esse exemplo. Nesse caso o projeto será copiado 
para o workspace e qualquer alteração nele não altera os arquivos originais

e) Em seguida compile todos os arquivos do projeto ("Rebuild all project files"). Para isso, selecione o projeto na janela "Project explorer" e então clique em "Project->build project".

A priori, não deve conter erro.


## Instalar STM32CubeMX na forma de plug-in para Eclipse

**STM32CubeMX** é uma ferramenta gráfica muito útil para configurar microcontroladores da família STM32. De fato, esses dispositivos possuem vários periféricos internos, e configurar 
sua inicialização diretamente no código é um processo complexo. Portanto, partir de um código inicial já ajustado é o que buscamos. Desde que sejam respeitadas algumas premissas, o código gerado pode 
ser alterado pelo STM32CubeMX, sem que se perca código do usuário.

Apesar de existir uma versão standalone do STM32CubeMX, é preferível instalar a versão que fica integrada ao Eclipse, do SW4STM32. Para sua instalação, basta seguir os 
procedimentos da seção 4.3 do arquivo "STM32CubeMX Eclipse plug-in for STM32 configuration and initialization C code generation.pdf" do link https://www.st.com/en/development-tools/stsw-stm32095.html


## Primeiro projeto (LED blink) com STM32CubeMX

Abra o SW4STM32, e, ao clicar no icone "Open Perspective" (lado superior à direita), uma das opções é a do STM32CubeMX. A seguir, basta seguir os procedimentos do video 
https://www.youtube.com/watch?v=YZjnCOun1wU , com as seguintes alterações:

a) Escolha o microcontrolador STM32F103C8T6. 

b) Não inicializar a USB. 

c) Coloque o pino PC13 com modo GPIO_Output

d) No períférico RCC, escolha "Crystal/Ceramic Ressonator" como fonte para High Speed Clock (HSE)

e) No períférico SYS, escolha "Serial Wire" como método de depuração

f) Para configurar o relógio, deve-se clicar na aba "Clock Configuration". Para operar na máxima frequencia (72MHz), e partindo de um relógio externo, é necessário usar o PLL para se obter essa frequencia. 
Como não é multiplo de 8MHz (HSE), então deve-se multiplicar por 9 no PLL. Observa-se que, se for usada a porta USB, que usa um relógio de 48MHz, deve-se buscar outra combinação de PLL e divisores.

g) Para o LED iniciar desligado, PC13 deve ser iniciado em nível alto. Isso deve ser feito na aba "Configuration", selecionando GPIO em System, e em seguida definir o estado do pino PC13.

h) Seguir o restante do procedimento do vídeo para gerar o código

i) No laço infinito de main.c, colocar o seguinte código, logo após "/* USER CODE BEGIN 3 */":

    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, 0); // Led acende;
    HAL_Delay(500);
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, 1); // Led apaga;
    HAL_Delay(500);

h) Fazer um Build no projeto. Não deve gerar erro.

i) Conectar a bluepill com o STLINK/V2 (GND, SWIO e SWCLK). Se a placa não for ligada na porta USB e se não tiver outra fonte de alimentação externa, pode conectar também 3,3V da STLINK/V2

j) Configurar no OpenOCD para lidar com o STLink/V2 ligado na bluepill (atenção, não alimentar a bluepill com os 3,3V do STLINK/V2 se a placa for ligada à porta USB). 
Configurar o depurador do modo Run de acordo com o video, ou seja, onde estiver no arquivo .cfg o comando "reset_config srst_only srst_nogate connect_assert_srst" trocar por "reset_config trst_only".

k) Executar Run

O Led da placa deve piscar.

O arquivo "bluepill_blink_stm32cube.rar" contém esse exemplo. Coloque seu conteúdo no workspace de trabalho e abra no Eclipse. 


## Recomendações

Devido à grande quantidade de periféricos, sugere-se usar o STM32CubeMX para sua configuração e inicialização. Se a configuação de hardware e relógio for similar à de outro projeto, em vez de criar um a partir do zero,
é melhor abrir um já existente com o STM32CubeMX e usar a opção "File -> Save As" para criar seu projeto a partir de um outro já funcional (blink, por exemplo)

Para novos projetos, sugere-se alterar o mínimo possível os arquivos gerados pelo STM32CubeMX. Percebe-se, ao ver o conteúdo de main.c, que há vários pontos em que é permitida a inclusão de código de usuários. 
Toda vez que o projeto precisar ser reconfigurado pelo STM32CubeMX, esse código será preservado (isso se "keep user code when re-generating" estiver ativado, que fica em "Project -> Settings -> Code Generator" do STM32CubeMX).
Logo, sudere-se criar seu próprio código num par instance.c/instance.h, sendo a função instance(), que é chamada em main.c, pouco antes do laço infinito, e que inicializa sua instância de trabalho e permanece num lado infinito.

Uma grande vantagem do STM32F103 é que ele já possui uma porta USB integrada operando em velocidade Full Speed (12Mbps), operando em vários modos. O fabricante ST possui um driver para transformar a porta USB numa porta COM serial
Basta baixar em http://www.st.com/en/development-tools/stsw-stm32102.html, ou na pasta "STM32 Virtual COM Port Driver" do SDK. O vídeo mostra como fazer isso com a blue pill: https://www.youtube.com/watch?v=YZjnCOun1wU

Se necessário, pode-se lançar uma ferramenta externa a partir do Eclipse. Um exemplo de uso desse recurso é lançar um terminal para porta serial. "Run -> External Tools -> External Tools COnfiguration". Cria-se um perfil PuTTY e
em "Location" coloca-se o local onde está o executável. Você verá na janela principal do Eclipse um ícone para lançar o Putty "Run PuTTY". O PuTTY pode ser baixado de http://www.putty.org/

Existe a possibilidade de se usar o STLink para depuração, em vez da porta USB. Assim, pode-se usar a função printf para direcionar mensagens para um console. 
Para evitar a chance de perder seu código

Na internet existem muitas fontes de informação para desenvolvimento com STM32. Embora boa parte faça uso de placas da ST, a "blue pill" está se disseminando bastante, devido ao baixíssimo preço. 
