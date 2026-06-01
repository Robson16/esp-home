# Checklist de Migração de Hardware: "Operação Pi Node 2"

## Fase 1: Desmontagem e Readequação Física

- [x] Desligar a energia completa do Cluster Pi (Master e Node 2).
- [x] Remover o ESP32 Node 2 e o Display OLED da caixa do cluster.
- [x] Conectar o BMP280 ao Pi Node 2:
  - Ligar VCC no pino 1 (3.3V) do Pi Node 2.
  - Ligar GND em qualquer pino terra (ex: Pino 6 ou 9).
  - Ligar SDA no pino 3 (GPIO 2 - SDA).
  - Ligar SCL no pino 5 (GPIO 3 - SCL).
- [x] Conectar os Coolers de Exaustão ao Pi Node 2: (Lembrando de manter os módulos relé ou transistores, pois o Pi manda 3.3V e os coolers são 5V).
  - Cooler Exaustor 1 (ligava a 35°): Conectar o sinal no GPIO 27 (Pino 13).
  - Cooler Exaustor 2 (ligava a 40°): Conectar o sinal no GPIO 22 (Pino 15).
- [x] Verificar se os coolers nativos das CPUs continuam conectados:
  - Pi Master: GPIO configurado para o cooler da CPU Master.
  - Pi Node 2: GPIO configurado para o cooler da CPU Node 2.

## Fase 2: Configuração Nativa do SO (Pi Node 2)

- [x] Ligar o cluster.
- [x] Acessar o pi-node-2 via SSH.
- [x] Habilitar o barramento I2C do hardware:
  - Rodar `sudo raspi-config`.
  - Ir em Interface Options -> I2C -> Enable.
- [x] Garantir que o cooler da própria CPU do Node 2 está configurado via Kernel:
  - Abrir `sudo nano /boot/firmware/config.txt`.
  - Confirmar a linha `dtoverlay=gpio-fan,gpiopin=17,temp=60000` (ou o pino/temp que você definiu para a CPU).

## Fase 3: A Ponte Python (O "Novo" ESPHome)

- [x] Instalar as bibliotecas Python necessárias no pi-node-2:

  ```bash
  sudo apt install python3-pip python3-smbus i2c-tools
  pip3 install paho-mqtt adafruit-circuitpython-bmp280 gpiozero
  ```

- [x] Detectar se o BMP280 está sendo lido pelo I2C:
  - Rodar `i2cdetect -y 1` e procurar pelo endereço 76 ou 77.
- [x] Escrever o script Python (cluster_monitor.py) que:
  - Lê o BMP280 via I2C.
  - Aplica a lógica if temp >= 35 (liga GPIO 27) e if temp >= 40 (liga GPIO 22).
  - Envia a temperatura e o estado dos coolers via MQTT para o broker 192.168.1.101.
- [x] Criar um serviço no systemd para garantir que esse script Python rode automaticamente em background toda vez que o Node 2 for reiniciado (igual o ESPHome fazia no chip).

## Fase 4: Ajustes no Home Assistant

- [x] Acessar os arquivos YAML do Home Assistant (ou a UI de Dispositivos MQTT).
- [x] Criar ou apontar os Sensores MQTT (Temperatura da Caixa) e Switches MQTT (Cooler 1 e 2) para os mesmos nomes de entidade que a sua dashboard já utiliza.
- [x] Validar na dashboard se a temperatura aparece e se os botões manuais ativam os exaustores pelo Python.

## Fase 5: Cirurgia no ESP32 Node 1 (O do Quarto)

- [x] Acessar o YAML do ESP32 Node 1 no ESPHome.
- [x] Calibrar as configurações de Hardware do DHT22 e do BMP280.
- [x] Compilar e instalar via OTA (Over The Air).

## Fase 6: Nova Vida para o ESP32 Node 2

- [x] Escrever um novo arquivo YAML básico no ESPHome para ele.
- [x] Configurar apenas o OLED, DHT22/BMP280.
- [ ] Espalhar pelo cômodo desejado com uma fonte de celular e conectar ao Wi-Fi.