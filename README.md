# Estação de Solda com Controle PID

Projeto de controlador de temperatura para estação de solda utilizando Arduino com FreeRTOS, controle de potência via TRIAC com sincronização de cruzamento por zero, e sensor MAX6675 para leitura de temperatura.

## 📋 Sumário

- [Versões Disponíveis](#versões-disponíveis)
- [Hardware Necessário](#hardware-necessário)
- [Pinagem e Conexões](#pinagem-e-conexões)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Operação](#operação)
- [Segurança](#segurança)
- [Troubleshooting](#troubleshooting)

---

## 🔄 Versões Disponíveis

### **sketch_mar02a_safe.ino** ⭐ RECOMENDADA
- Versão original com **limite de segurança de 450°C**
- Corte automático de potência ao atingir limite
- Alerta visual no display
- **Ideal para uso em projeto**

### **sketch_mar02a_enhanced.ino** 🚀 AVANÇADA
- Todas as funcionalidades da versão Safe
- **Sincronização de variáveis** (elimina race conditions)
- **Detecção de falha do sensor** (open circuit, short circuit)
- **Watchdog timer** (timeout de segurança)
- **Reset automático de integral** do PID
- **Validação de leituras** com limites máximos
- **Estados do sistema** (NORMAL, OVERHEAT, SENSOR_ERROR)
- **Melhor debounce** de botão
- **Ideal para produção**

---

## 🛠️ Hardware Necessário

### Componentes Principais
| Item | Quantidade | Observação |
|------|-----------|-----------|
| Arduino Uno/Mega | 1 | Compatível com FreeRTOS |
| MAX6675 Thermocouple Module | 1 | Com sensor tipo K |
| Display OLED SSD1306 128x64 | 1 | I2C, 0x3C padrão |
| TRIAC BTA16-600B | 1 | 16A, 600V (ou equivalente) |
| MOC3021 Optoisolador | 1 | Para disparo do TRIAC |
| Resistor 180Ω 2W | 1 | Gate do TRIAC |
| Resistor 100Ω | 1 | LED indicador (opcional) |
| LED 5mm vermelho | 1 | Indicador de saída (opcional) |
| Botão com mola | 1 | Tátil 12mm |
| Encoder Rotativo | 1 | Opcional (futuro) |
| Fusível 16A + Suporte | 1 | Proteção de circuito |
| Transformador 220V→24V | 1 | Para alimentação (opcional) |

---

## 📌 Pinagem e Conexões

### **Tabela de Pinos**

| Pino Arduino | Função | Conexão | Observação |
|-------------|--------|---------|-----------|
| **PIN 2** | INT0 / Zero-Cross | Detector AC | 5V tolerante |
| **PIN 3** | INT1 / Encoder A | Encoder (futuro) | 5V tolerante |
| **PIN 4** | GPIO / Encoder B | Encoder (futuro) | 5V tolerante |
| **PIN 5** | GPIO / Botão | Botão com GND | Pullup interno |
| **PIN 9** | PWM / MOC Gate | Optoisolador | PWM @ 16kHz |
| **PIN 10** | SPI CS | MAX6675 | Chip Select |
| **PIN 12** | SPI MISO | MAX6675 | Data In |
| **PIN 13** | SPI SCK | MAX6675 | Clock |
| **PIN A4** | I2C SDA | Display OLED | Pullup 4.7kΩ |
| **PIN A5** | I2C SCL | Display OLED | Pullup 4.7kΩ |
| **GND** | Ground | Referência | Comum |
| **5V** | Power | Lógica | ~500mA máx |

### **1. Display OLED SSD1306 (I2C)**

```
Display OLED          Arduino
────────────────────────────
GND  ─────────────── GND
VCC  ─────────────── 5V
SCL  ─────────────── A5 (SCL)
SDA  ─────────────── A4 (SDA)
```

### **2. MAX6675 Thermocouple Amplifier (SPI)**

```
MAX6675          Arduino
──────────────────────────
GND  ─────────── GND
VCC  ─────────── 5V
SCK  ─────────── PIN 13 (SPI SCK)
CS   ─────────── PIN 10 (SPI CS)
SO   ─────────── PIN 12 (SPI MISO)

Sensor Tipo K
──────────────
Vermelho (+)  ─── Terminal +
Amarelo (-)   ─── Terminal -
```

### **3. Zero-Crossing Detector (Sincronismo AC)**

```
Detector ZC          Arduino
──────────────────────────
GND  ─────────────── GND
+5V  ─────────────── 5V
OUT  ─────────────── PIN 2 (INT0)
```

### **4. Optoisolador MOC3021 (Disparo TRIAC)**

```
MOC3021                  Arduino
─────────────────────────────
Pin 1 (Anode)   ─── PIN 9 (via Resistor 180Ω)
Pin 2 (Cathode) ─── GND
Pin 5 (GND)     ─── GND
Pin 6 (MT2)     ─── TRIAC Gate
Pin 7 (MT1)     ─── Carga AC
```

### **5. TRIAC BTA16-600B (Controle de Carga AC)**

```
TRIAC BTA16-600B        Circuito
──────────────────────────────────
MT2 (Gate)   ─── MOC3021 Pin 6
MT1 (A1)     ─── Neutro (N) da carga
A2 (MT1/A2)  ─── Fase (L) da carga
```

### **6. Botão de Controle**

```
Botão      Arduino
──────────────────
Pin 1 ──── PIN 5
Pin 2 ──── GND
```

---

## 💾 Instalação

### 1. Preparar Arduino IDE

```
Instalar bibliotecas (Sketch → Include Library):
- Arduino_FreeRTOS_Library
- EEPROM (padrão)
- max6675
- Wire (padrão)
- Adafruit_GFX
- Adafruit_SSD1306
```

### 2. Selecionar Placa e Porta

```
Tools → Board → Arduino Uno (ou Mega)
Tools → Port → COM3 (ou equivalente)
```

### 3. Upload

```
Sketch → Upload (Ctrl+U)
```

---

## ⚙️ Configuração

### Ajustar Frequência AC (50Hz ou 60Hz)

```cpp
// Versão atual: 60Hz
#define HALF_CYCLE_TICKS 16666

// Para 50Hz:
#define HALF_CYCLE_TICKS 20000
```

### Ajustar Ganhos do PID

```cpp
float Kp=18, Ki=0.08, Kd=90;
```

---

## ▶️ Operação

### Inicialização
1. Conectar estação ao AC 220V/110V
2. Arduino liga e exibe "INIT..." no display
3. Aguardar 2-3 segundos até estabilizar

### Uso Normal
- **Display mostra:** Temperatura atual, setpoint, potência
- **Botão (PIN 5):** Aumenta setpoint em 5°C (máx 450°C)

### Estados do Sistema

#### ✅ NORMAL
Temperatura entre 150°C e 450°C - Sistema funcionando corretamente

#### ⚠️ OVERHEAT (apenas Enhanced)
Temperatura ≥ 450°C - Potência cortada automaticamente

#### 🔴 SENSOR_ERROR (apenas Enhanced)
Leitura inválida - Potência = 0%, display mostra erro

---

## 🔒 Segurança

### ⚠️ AVISOS IMPORTANTES

1. **ALTA TENSÃO AC 220V/110V**
   - Nunca tocar em componentes AC sem desligar
   - Usar encapsulamento isolante
   - Usar EPI apropriado

2. **Limite de Temperatura 450°C**
   - Proteção automática ao atingir limite
   - Elemento aquecedor pode danificar se > 500°C

3. **Corrente AC**
   - TRIAC BTA16-600B máx 16A @ 600V
   - Para correntes maiores, usar dissipador térmico
   - Fusível de 16A protege circuito

4. **Sensor Thermocouple**
   - Manter afastado de AC
   - Não permitir curto-circuito nos terminais

### Checklist de Segurança

- [ ] Circuito AC isolado em encapsulamento
- [ ] Sensor thermocouple longe da AC
- [ ] Todos os fios dimensionados adequadamente
- [ ] Fusível 16A instalado
- [ ] TRIAC com radiador se I > 8A
- [ ] Aterramento adequado
- [ ] Teste com carga baixa antes de usar

---

## 🐛 Troubleshooting

### Display não aparece
```
Verificar I2C (endereço padrão 0x3C)
Se necessário, ajustar para 0x3D
```

### Temperatura não sobe
Verificar:
1. PIN 9 com osciloscópio
2. Detecção de zero-crossing em PIN 2
3. Continuidade TRIAC-Carga

### Temperatura oscila demais
Ajustar PID:
```cpp
float Kp=12;  // Reduzir
float Kd=120; // Aumentar
```

### Sensor lê valores aleatórios
1. Torcer fios do sensor
2. Adicionar capacitor 0.1µF no CS
3. Afastar sensor de fios AC

---

## 📊 Especificações Técnicas

| Parâmetro | Valor |
|-----------|-------|
| Temperatura Min | 150°C |
| Temperatura Max | 450°C |
| Resolução | ±2°C |
| Tempo de controle | 200ms |
| Frequência PID | 5 Hz |
| Frequência AC | 50/60 Hz (configurável) |
| Potência saída | 0-2kW |
| Proteção | Limite 450°C, Fusível 16A |

---

**Última atualização:** 2026-03-02