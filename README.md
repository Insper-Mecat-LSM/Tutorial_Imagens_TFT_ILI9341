# Tutorial_Imagens_TFT_ILI9341
Tutorial para a inserção de curvas, imagens e animações no display tft de controlador ILI9341 utilizando biblioteca Adafruit GFX e MCUFRIEND

Agora que você já seguiu os passos para obtenção das bibliotecas necessárias para funcionamento do display e conheceu algumas funções básicas como a criação de linhas, retângulos, círculos e triângulos, é hora de explorarmos algumas ideias envolvendo imagens, animações e o traçado de curvas que a biblioteca permite. 

Uma das estratégias mais comuns para simular movimentação de elementos sobre a tela é a dinâmica entre aceso e apagado. Um bom exemplo dessa estratégia são os jogos antigos no estilo Snake em que, à medida que o objeto avança e novos elementos são adicionados à frente, para manter o comprimento, o último elemento do conjunto é sobreposto por uma camada de mesma cor daquela de fundo, dando a impressão de que o objeto se move pela tela. 

<div align="center">
<img src ="https://user-images.githubusercontent.com/72100554/233463964-897fa1e6-667f-407f-814a-ad64f1fe6663.gif" width="300px"/>
</div>

O código abaixo mostra a implementação dessa ideia. À medida que a variável é aumentada, um novo retângulo é exibido ligeiramente à frente e o imediatamente anterior é coberto por outro com a mesma cor de fundo. 

```C++
// Bibliotecas
#include "Arduino.h"
#include "mbed.h"
#include <MCUFRIEND_kbv.h>

MCUFRIEND_kbv tft;

// Orientação do Display
uint8_t Orientation = 1; // Paisagem

// Tabela de Cores
#define BLACK 0x0000
#define BLUE 0x001F

// Variáveis
int y = 0;

// Função para desenhar Snake
void snake() {
  //Desenha o retângulo azul.
  tft.fillRoundRect(150, 140 - y, 10, 60, 0, BLUE);
  //'Apaga' com retângulo preto. Note que ele está deslocado de 60 px, que é o comprimento do retângulo azul.
  tft.fillRoundRect(150, 200 - y, 10, 40, 0, BLACK);
}

void setup(void) {
  tft.reset();
  tft.begin();
  tft.setRotation(Orientation);
  tft.fillScreen(BLACK); // Fundo do Display
}

void loop() {
  // Desenha, incrementa o valor de 'y' e espera 15 ms.
  snake();
  y++;
  wait_ms(15);
}
```

Nesse caso a movimentação ocorre em uma única direção. Perceba que um retângulo preto é criado logo atrás do retângulo em azul. Experimente alterar a cor de fundo do display para WHITE e veja o que está acontecendo.

## Curvas 

<div align="center">
<img src ="https://user-images.githubusercontent.com/72100554/233660648-4f44eef9-420e-4a91-9b86-d21d4348e6c9.gif" width="300px"/>
</div>



Curvas podem ser desenhadas com o auxílio de funções matemáticas. Portanto elipses, hipérboles, parábolas, catenárias, senoides etc. podem ter seus pontos calculados por expressões matemáticas e, a cada iteração, um novo pixel é desenhado na tela. Vejamos o exemplo.

```C++
//Bibliotecas
#include "mbed.h"
#include "Arduino.h"
#include <MCUFRIEND_kbv.h>
MCUFRIEND_kbv tft;
#include <math.h>

//Orientação do Display
uint8_t Orientation = 1; //Paisagem

//Tabela de Cores
#define BLACK   0x0000
#define GREEN   0x07E0
#define YELLOW  0xFFE0

//Variáveis
int x,y=0;
int x_0 = 5;
int y_0 = 100;
int A = 40;

int a = 5;
int b = 4;

//Criando funções
void seno (){
    //y =y0+ A*seno(6*x*0.0174533[grau para radiano])
    tft.drawPixel(x_0+x, y_0+A*sin(6*x*0.0174533), GREEN);
}

void cosseno (){
    //y =y0+ A*cosseno(6*x*0.0174533[grau para radiano])
    tft.drawPixel(x_0+x, y_0+A*cos(6*x*0.0174533), YELLOW);
}


void setup(void)
{
    tft.reset();
    tft.begin();
    tft.setRotation(Orientation);
    tft.fillScreen(BLACK);  // Fundo do Display

    //y = A*sin(x*0.0174533);

}

void loop() 
{
seno();
//cosseno();
x++;

wait_ms(5);
}
```

Caso deseje alterar a espessura da curva, busque criar offsets redesenhando a mesma curva alguns pixels para cima ou para baixo e verifique o resultado. Altere os parâmetros dentro da função para variar o período.  O que acontece caso troque de ```drawPixel()``` para ```fillRoundRect()``` ? 
Reduza os retângulos de tamanho e aumente o raio de arredondamento. Se quiser traçar uma linha entre os pontos calculados, utilize a função ```drawLine()```.

Note que essa mesma funcionalidade pode traçar o estado de algum pino, gerando um gráfico no tempo ou associado a qualquer outra variável de interesse.

## Imagens

<div align="center">
<img src ="https://user-images.githubusercontent.com/72100554/233661639-43cedef1-ebc7-4132-a5a3-1d4891e89067.jpeg" width="300px"/>
</div>


A importação de imagens através da função drawBitmap() não ocorre de forma direta para o microcontrolador. Antes, é preciso convertê-la em uma matriz que possui os hexadecimais correspondentes a cada pixel da imagem. O autor obteve sucesso ao utilizar o site: http://www.rinkydinkelectronics.com/t_imageconverter565.php. 

Nele, basta importar uma imagem nos formatos ```.png```, ```.jpg``` ou ```.gif``` com as dimensões desejadas (lembre-se que seu display possui resolução de 320x240) e clicar em ```Make file```. A seguir basta realizar download do arquivo ```.c```. Para extrair a matriz gerada pelo site, abra o arquivo com o bloco de notas e copie tudo que estiver dentro das chaves de ```PROGMEN```, segue código.

```C++
#include "mbed.h"
#include "Arduino.h"
#include <MCUFRIEND_kbv.h>

//#include "logo.h" //Descomente esta linha para utilizar o arquivo .h

MCUFRIEND_kbv tft;

//Orientação do Display
uint8_t Orientation = 1; //Paisagem

//Cores
#define BLACK   0x0000
#define WHITE   0xFFFF

void setup(void)
{
    tft.reset();
    tft.begin();
    tft.setRotation(Orientation);
    tft.fillScreen(WHITE);  // Fundo do Display
    
    //Cole os hexadecimais dentro das chaves abaixo. 
    //Caso use o arquivo .h, delete a linha abaixo e crie o arquivo .h
    const uint16_t  PROGMEM logo[] = {};
  
    //Parâmetros da função: posições x,y da imagem, array 'logo' e as dimensões da imagem
    tft.drawRGBBitmap(0,45, logo, 320, 129);//Insper    
}


void loop()
{   
    
}

```

Procure substituir os hexadecimais existentes, ajustando os parâmetros necessários (dimensões) e verifique se a imagem é carregada corretamente. Note que eventuais imperfeições deverão ser corrigidas diretamente na matriz que foi gerada pelo site, substituindo o hexadecimal pelo desejado. 

Para remover essa grande quantidade de hexadecimais do ```main```, podemos criar um arquivo ```.h``` e o referenciar no ```main```, como foi feito no código acima, bastando descomentar a linha que referencia o ```.h``` e remover a linha que cria a ```logo[]```. Portanto crie um arquivo ```logo.h``` dentro da pasta do seu projeto e o redija dessa forma:

```C++
//Arquivo .h 
#include "mbed.h"
#ifndef logo_h
#define logo_h

//Cole os hexadecimais dentro das chaves abaixo
const uint16_t  PROGMEM logo[] = {};

#endif
```

## Animações 

<div align="center">
<img src ="https://user-images.githubusercontent.com/72100554/233665023-ecd263b9-e20e-4acc-9e7c-0deee5c35a2c.gif" width="300px"/>
</div>


Para animações, a mesma ideia de sobreposição pode ser usada. A diferença é que ao invés de utilizarmos formas ou bitmaps estáticos, sobrescreveremos imagens ligeiramente diferentes, dando a ideia de animação. O autor recomenda a utilização de ‘sprites’, assim como você pode ter utilizado em DeSoft. Salve-os em um arquivo ‘.h’ e use-os sequencialmente, intercalados por uma forma (retângulo, triângulo, círculo) de mesma dimensão do sprite. Caso deseje movimentá-lo pela tela, siga a mesma ideia da seção 1 do ‘Snake’. Vamos ao exemplo:

```C++

#include "mbed.h"
#include "Arduino.h"
#include <MCUFRIEND_kbv.h>

#include "sprites.h"

MCUFRIEND_kbv tft;

//Orientação do Display
uint8_t Orientation = 1; //Paisagem

//Cores
#define BLACK   0x0000
#define WHITE   0xFFFF

int x=0;

void espera(){
    wait_ms(50);
}

void setup(void)
{
    tft.reset();
    tft.begin();
    tft.setRotation(Orientation);
    tft.fillScreen(BLACK);  // Fundo do Display

    
}

void loop()
{   
    tft.drawRGBBitmap(80+x,100, frame_0, 40, 40);//Frame 0/12
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_1, 40, 40);//Frame 1/12
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_2, 40, 40);//Frame 2/12
    espera();
    x+=2; 
    tft.drawRGBBitmap(80+x,100, frame_3, 40, 40);//Frame 3/12   
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_4, 40, 40);//Frame 4/12  
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_5, 40, 40);//Frame 5/12 
    espera();
    x+=2; 
    tft.drawRGBBitmap(80+x,100, frame_6, 40, 40);//Frame 6/12   
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_7, 40, 40);//Frame 7/12  
    espera();
    x+=2; 
    tft.drawRGBBitmap(80+x,100, frame_8, 40, 40);//Frame 8/12  
    espera();
    x+=2; 
    tft.drawRGBBitmap(80+x,100, frame_9, 40, 40);//Frame 9/12   
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_10, 40, 40);//Frame 10/12   
    espera();
    x+=2;
    tft.drawRGBBitmap(80+x,100, frame_11, 40, 40);//Frame 11/12
    espera(); 
    x+=2;   

}

```

Infelizmente a biblioteca não possui a opção de múltiplas camadas, de modo que não se torna tão trivial sobrescrever as imagens sem danificar a imagem de fundo.

Atenção: Para altas taxas de atualização da tela, recomenda-se utilizar imagens pequenas, 40x40 pixels por exemplo, para que a imagem seja rapidamente desenhada. 
Sempre tenha em mente o propósito do embarcado: imagens grandes consomem bastante memória e não são impressas na tela de forma instantânea aos olhos.

Se desejar realizar animações em que haja rotação da forma ou da imagem, busque por matriz de rotação. A eletiva de Visão de Máquina trará ferramental que auxiliará nessa tarefa.
