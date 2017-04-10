---
layout: post
title:  "Controlando um braço robótica com Arduino e Leap Motion"
date:   2016-08-16 22:00:00 -0300
categories: arduino leap robotics
---

Esta brincadeira visa controlar um pequeno braço robótico (MeArm) utilizando os movimentos das mãos. Em um primeiro momento parece algo meio complicado, mas na verdade a API do Leap Motion torna o trabalho bem simples. Eu gravei o vídeo abaixo mostrando o funcionamento do projeto final.

<iframe width="560" height="315" src="https://www.youtube.com/embed/bUOMn1RQ4Mo" frameborder="0" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/0NjoCN_xS8g" frameborder="0" allowfullscreen></iframe>


### Partes mecânicas

O projeto da garra não foi desenvolvido por mim, [foi obtido do Thingsverse e chama-se MeArm](http://www.thingiverse.com/thing:360108). A garra tem uma construção bastante simples e pode ser cortada a laser em acrílico ou MDF. A que eu fiz foi em MDF, mas sugiro que faça em acrílico pois algumas partes pequenas acabaram ficando frágeis em MDF.


### Ligações eletrônicas

O projeto eletrônico também é bastante simples. A garra usa 4 servos que permitem movimentar o braço em 3 dimensões de liberadade e mais a pinça. Segue abaixo as ligações dos servos no Arduino. 

<iframe frameborder='0' height='448' marginheight='0' marginwidth='0' scrolling='no' src='https://circuits.io/circuits/2420483-mearm/embed#breadboard' width='650'></iframe>

### Leap Motion

Para quem não conhece, o [Leap Motion](leapmotion.com) é um pequeno dispositivo que permite controlar o computador e jogos a partir do movimento das mãos. Funciona como um mini kinect, com uma área de ação reduzida porém, muito mais preciso. Abaixo um vídeo de demonstração. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/zXghYjh6Gro" frameborder="0" allowfullscreen></iframe>

A Leap fornece um SDK com uma API simples de usar e uma excelente documentação. O SDK está disponível para diversas plataformas e linguagens. Nesse projeto eu utilizei C#.

### Comunicação com o Arduino - FIRMATA

O Leap motion necessita de uma capacidade considerável de processamento para fazer o reconhecimento das mãos e dos gestos. O Arduino possui um microcontrolador de 16Mhz, não sendo possível processar tudo nele. Este projeto utiliza um computador para controlar o Arduino. Para fazer esse controle remoto a partir do computador foi utilizado o protocolo FIRMATA. O FIRMATA é um protocolo de comunicação feito em cima da comunicação serial (a comunicação normal Serial.println utilizada para debug no Arduino). Ao invés de criarmos a nossa própria estratégia para envio e recebimento de informações, vamos usar um padrão comum.

A parte legal é que o FIRMATA possui diversas bibliotecas para muitas linguagens, nesse projeto foi utilizada a biblioteca [Arduino4Net](https://www.nuget.org/packages/Arduino4Net/0.0.2).

### Código

O Leap Motion possui o seguinte sistema de coordenadas:

{<2>}![Leap Motion Coordenate](https://di4564baj7skl.cloudfront.net/documentation/v2/images/Leap_Axes.png)

Os eixos foram utilizados da seguinte maneira:

Eixo X: Controla a base do braço (movimentos para os lados)
Eixo Y: Controla a altura do braço (movimentos para cima e para baixo)
Eixo Z: Controla a distância do braço (movimentos para frente e para trás)

Além das posições da mão, o leap motion também permite identificar gestos (movimentos circulares, abrir e fechar a mão, toques em uma superfície e etc). Aqui foi utilizado o reconhecimento de gestos de pinça para abrir e fechar a garra.

Agora que todos os componentes estão explicados, vamos ao código! :D

#### No Computador
A primeira classe é responsável por fazer a comunicação com o Arduino utilizando o FIRMATA (biblioteca Arduino4Net). 
	
	public class MeArm
    {
        private Arduino arduino;

        private int pinServo1;
        private int pinServo2;
        private int pinServo3;
        private int pinServo4;

        public MeArm(String port)
        {
            //Define as portas nas quais
            //os servos estão ligados
            pinServo1 = 2;
            pinServo2 = 3;
            pinServo3 = 4;
            pinServo4 = 5;
            try
            {
                //Inicializa o Arduino e as portas como PWM
                arduino = new Arduino(port);
                arduino.PinMode(pinServo1, PinMode.Servo);
                arduino.PinMode(pinServo2, PinMode.Servo);
                arduino.PinMode(pinServo3, PinMode.Servo);
                arduino.PinMode(pinServo4, PinMode.Servo);
            }
            catch(Exception ex)
            {
                Console.WriteLine("Falha ao conectar no arduino");
            }
            
        }
        public void MoverBase(int degree)
        {
            Console.WriteLine("Base: {0} graus", degree);
            arduino.AnalogWrite(pinServo3, degree);
        }

        public void MoveArm1(int degree)
        {
            Console.WriteLine("Arm1: {0} graus", degree);
            arduino.AnalogWrite(pinServo4, degree);
        }

        public void MoveArm2(int degree)
        {
            Console.WriteLine("Arm2: {0} graus", degree);
            arduino.AnalogWrite(pinServo2, degree);
        }

        public void MoverGarra(int degree)
        {
            //Limite para que a garra não abra demais
            if (degree < 110)
            {
                degree = 110;
            }
            Console.WriteLine("Movendo garra: {0} graus", degree);
            arduino.AnalogWrite(pinServo1, degree);
        }
    }

A classe é bastante simples, apenas inicializa o Arduino e as portas como PWM e define alguns métodos para utilizar na manipulação dos ângulos dos servos.

Na segunda classe é onde a "mágica" acontece. Na verdade é bastante simples também.

	public class LeapListener : Listener
    {
        private MeArm meArm;

        public override void OnInit(Controller controller)
        {
            //Altere para a sua porta COM
            meArm = new MeArm("COM8");
            base.OnInit(controller);
        }

        public override void OnConnect(Controller controller)
        {
            Console.WriteLine("Conectado");
        }

        //Not dispatched when running in debugger
        public override void OnDisconnect(Controller controller)
        {
            Console.WriteLine("Desconectado");
        }

        public override void OnFrame(Controller arg0)
        {
            Frame frame = arg0.Frame();
            Console.WriteLine("Hands: " + frame.Hands.Count);

            //Verifica se existe uma mão na frente do Leap Motion
            if(frame.Hands.Count > 0)
            {
                //Seleciona a primeira mão 
                //já que o leap pode capturar até duas mãos
                //na mesma cena.
                Hand hand = frame.Hands[0];

                //Captura o valor correspondente a rotação do braco
                float x = hand.PalmPosition.x;
                //Essa função Map faz a mesma coisa que a 
                //função map existente no Arduino, ela mapeia um 
                //valor entre um intervalo para o valor correspondente
                //em um diferente intervalo
                float angle1 = Util.Map(x, -240, 240, 0, 180);
                Console.WriteLine("X: " + x);
                Console.WriteLine("Angle 1: " + angle1);
                meArm.MoverBase(Convert.ToInt32(angle1));

                //Captura a posição correspondente
                //a altura do braço
                float y = hand.PalmPosition.y;

                //Limita o valor máximo para evitar de ter
                //que mover muito o braço para cima e para 
                //baixo para mover o braço do robô
                if (y > 250)
                    y = 250;

                float angle2 = Util.Map(y, 0, 250, 0, 180);
                Console.WriteLine("Y: " + y);
                Console.WriteLine("Angle 2: " + angle2);
                //Inverte o valor para corresponder aos movimentos 
                //reais (mais longe sobre e mais perto desce)
                angle2 = (angle2 - 180) * -1;
                meArm.MoveArm2(Convert.ToInt32(angle2));

                //Captura a posição de distância do braço
                float z = hand.PalmPosition.z;
                float angle3 = Util.Map(z, -260, 210, 0, 180);
                Console.WriteLine("Z: " + z);
                Console.WriteLine("Angle 3: " + angle3);
                meArm.MoveArm1(Convert.ToInt32(angle3));

                //Reconhece o gesto de pinça 
                //0 - Dedos abertos
                //1 - Dedos fechados
                float grab = hand.PinchStrength;
                Console.WriteLine("Grab: " + grab);
                float grabAngle = Util.Map(grab, 0, 1, 0, 180);
                Console.WriteLine("Grab angle: " + grabAngle);
                //Inverte para corresponder aos movimentos reais
                //dedos abertos abrem a garra e dedos fechados
                //fecham a garra
                grabAngle = (grabAngle - 180) * -1;
                meArm.MoverGarra(Convert.ToInt32(grabAngle));
            }
        }
    }

Primeiramente a classe inicializa um objeto do tipo MeArm passando a porta de comunicação (altere essa porta para a porta correspondente no seu computador).

Essa classe herda de Listener, uma classe do SDK do Leap Motion. Dessa forma os métodos OnInit, OnConnect, OnDisconnect e OnFrame são automaticamente chamados pelo SDK quando estes eventos ocorrerem. 

O método mais importante é o OnFrame, nele acontece a captura das informações necessárias para controlar o braço. Comentei o código com as informações de funcionamento.

Criei uma classe chamada Util com a função Map com o funcionamento semelhante a função [map existente no Arduino](https://www.arduino.cc/en/Reference/Map). Ela facilita a conversão de valores entre diferentes intervalos.

	public class Util
    {
        public static float Map(float x, float in_min, float in_max, float out_max, float out_min)
        {
            return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
        }
    }

E por último o Main para conectar ao Leap e configurar o Listener

	class Program
    {
        static void Main(string[] args)
        {
            LeapListener listener = new LeapListener();
            Controller leap = new Controller();
            leap.AddListener(listener);
            while (true) { }
        }
    }
    
O código completo do projeto está no [github e pode ser baixado aqui](https://github.com/guidefreitas/LeapArm).

#### No Arduino
No Arduino basta carregar o sketch StandardFirmata e pronto.

Abraço e até a próxima!