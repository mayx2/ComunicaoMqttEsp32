
# **Comunicação Bidirecional entre Dois ESP32 usando MQTT**

Este projeto implementa comunicação bidirecional entre **duas placas ESP32**, cada uma com:

* 1 **botão**
* 1 **LED**

Quando o botão da placa 1 é pressionado, **o LED da placa 2** acende.
Quando o botão da placa 2 é pressionado, **o LED da placa 1** acende.

A comunicação ocorre via **Wi-Fi Station** e **MQTT** usando o broker público HiveMQ.

---

# **1. Inicialização do sistema**

No `app_main()` o ESP32:

1. Inicializa NVS, rede (`esp_netif_init()`) e event loop.
2. Conecta ao Wi-Fi Station com `wifi_init_sta()`.
3. Configura GPIOs:

   * LED (pino 21) como saída
   * Botão (pino 23) como entrada com pull-up
4. Inicia o cliente MQTT.
5. Cria a task que monitora o botão.

---

# **2. Configuração dos pinos**

`configure_led()`:

* Reseta pinos
* LED como saída
* Botão como entrada com pull-up (botão no GND)

Estado do botão:

* `1` → solto
* `0` → pressionado

`led_set_state()` liga/desliga o LED.

---

# **3. Publicação do estado do botão (botao_task)**

A task:

1. Lê o botão continuamente.
2. Detecta mudança de estado.
3. Publica em **"Esp1/Botao"**:

   * `"0"` → apertado
   * `"1"` → solto
4. Só envia se o cliente MQTT estiver conectado.

Publica apenas quando o estado realmente muda.

---

# **4. Comunicação MQTT – Subscrição e Recebimento**

Ao conectar (`MQTT_EVENT_CONNECTED`), o ESP32 assina **"Esp2/Botao"**.

Quando chega uma mensagem (`MQTT_EVENT_DATA`):

* Verifica o tópico
* Se for `"Esp2/Botao"`:

  * `'0'` → liga o LED
  * `'1'` → desliga o LED

Assim cada ESP controla o LED da outra placa.

---

# **5. Ciclo de funcionamento**

1. ESP1 aperta o botão
   → publica `"0"` em `"Esp1/Botao"`
   → ESP2 recebe
   → ESP2 liga o LED

2. ESP2 aperta o botão
   → publica `"0"` em `"Esp2/Botao"`
   → ESP1 recebe
   → ESP1 liga o LED

Mesma lógica para soltar o botão.

---

# **6. Broker MQTT utilizado**

```
mqtt://broker.hivemq.com:1883
```

Broker público sem autenticação.

---

# **7. Estrutura geral do código**

### ✔ Conexão Wi-Fi

`wifi_init_sta()` no `wifi.c`.

### ✔ MQTT

Usa `esp-mqtt`.

### ✔ Publicação

`esp_mqtt_client_publish()` em `"Esp1/Botao"`.

### ✔ Subscrição

`esp_mqtt_client_subscribe()` em `"Esp2/Botao"`.

### ✔ LEDs

GPIO 21.

---

## **Funcionamento entre as duas placas ESP**

Ambas usam o mesmo código; só muda o **tópico MQTT**.

### ✔ ESP-1

* Publica: `Esp1/Botao`
* Assina: `Esp2/Botao`

### ✔ ESP-2

* Publica: `Esp2/Botao`
* Assina: `Esp1/Botao`

Cada ESP publica seu tópico e assina o do outro.

---

##  **Configurando o Wi-Fi**

No `wifi.c`, basta alterar:

```c
#define WIFI_SSID "NomeDaSuaRede"
#define WIFI_PASS "SenhaDoSeuWifi"
```

Ambos os ESP conectam automaticamente.

---

# **Funcionamento Resumido do Projeto**

Para gravar e rodar no ESP32:

1. Escolha a **porta COM**.
2. Faça o **Flash**.
3. Abra o **Monitor** na mesma porta.

Para gravar **no outro ESP**, basta **trocar a porta COM** e gravar novamente.

---


# **Resumo final**

O projeto cria comunicação entre duas placas ESP32 usando Wi-Fi Station e MQTT. Cada placa lê seu botão, envia o estado ao broker HiveMQ e, ao receber a mensagem da outra placa, atualiza seu LED local. Usa tasks FreeRTOS para leitura do botão, eventos MQTT para recebimento de mensagens e comunicação assíncrona eficiente.

---
