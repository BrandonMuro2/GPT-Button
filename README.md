# GPT
![](https://www.tijuana.tecnm.mx/wp-content/uploads/2014/11/Heading-Ing-sistemas-768x252.png)
# Depto de Sistemas y Computación
# Ing. En Sistemas Computacionales
# SISTEMAS PROGRAMABLES 
# Autor (es): Muro Andrade Brandon Arturo
# Repositorio:  GPT Button
# Fecha de revisión:  
# Objetivo: Desplegar en una oled una respuesta de chatgpt

# CÓDIGO
```python
# Despliegue de prompt de ChatGPT usando un botón
# Muro Andrade Brandon Arturo 20211818
# Tecnológico Nacional de México ITT
# Sistemas programables


import json
import network
import time
import urequests
import ssd1306

# Internal libs
i2c = machine.I2C(0, sda=machine.Pin(0), scl=machine.Pin(1), freq=400000)
oled = ssd1306.SSD1306_I2C(128, 64, i2c)
button = machine.Pin(14, machine.Pin.IN, machine.Pin.PULL_UP)


def chat_gpt(ssid, password, endpoint, api_key, model, prompt, max_tokens):
    """
        Description: This is a function to hit chat gpt api and get
            a response.
        
        Parameters:
        
        ssid[str]: The name of your internet connection
        password[str]: Password for your internet connection
        endpoint[str]: API enpoint
        api_key[str]: API key for access
        model[str]: AI model (see openAI documentation)
        prompt[str]: Input to the model
        max_tokens[int]: The maximum number of tokens to
            generate in the completion.
        
        Returns: Simply prints the response
    """
    # Just making our internet connection
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)
    
    # Wait for connect or fail
    max_wait = 10
    while max_wait > 0:
      if wlan.status() < 0 or wlan.status() >= 3:
        break
      max_wait -= 1
      print('waiting for connection...')
      time.sleep(1)
    # Handle connection error
    if wlan.status() != 3:
       print(wlan.status())
       raise RuntimeError('network connection failed')
    else:
      print('connected')
      print(wlan.status())
      status = wlan.ifconfig()
    
    ## Begin formatting request
    headers = {'Content-Type': 'application/json',
               "Authorization": "Bearer " + api_key}
    data = {"model": model,
            "prompt": prompt,
            "max_tokens": max_tokens}
    
    print("Attempting to send Prompt")
    r = urequests.post("https://api.openai.com/v1/{}".format(endpoint),
                       json=data,
                       headers=headers)
    
    if r.status_code >= 300 or r.status_code < 200:
        print("There was an error with your request \n" +
              "Response Status: " + str(r.text))
    else:
        print("Success")
        response_data = json.loads(r.text)
        completion = response_data["choices"][0]["text"]
        print(completion)
        
        # Mostrar 'completion' en la pantalla OLED con saltos de línea
        oled.fill(0)  # Borra la pantalla

        # Separar el texto 'completion' en líneas de máximo 21 caracteres
        completion_text = completion
        max_chars_per_line = 15
        lines = [completion_text[i:i+max_chars_per_line] for i in range(0, len(completion_text), max_chars_per_line)]

        # Mostrar las líneas en la pantalla
        for i, line in enumerate(lines):
            oled.text(line, 0, i * 10)  # Usar un espaciado vertical de 10 píxeles

        oled.show()

    r.close()

def button_callback(p):
    if not button.value():
        print("Button pressed. Sending prompt...")
        # Mostrar "Sending prompt" en la pantalla OLED
        oled.fill(0)  # Borra la pantalla
        oled.text("Sending prompt", 0, 0)
        oled.show()
        
        
        chat_gpt("INFINITUM6D7B",
                 "3405375900",
                 "completions",
                 "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",
                 "text-davinci-003",
                  "Recommend a book, respond in 10 words.",
                 20)

# Configura una interrupción en el botón para llamar a button_callback cuando se presiona el botón
button.irq(trigger=machine.Pin.IRQ_FALLING, handler=button_callback)

while True:
    # Tu código principal puede continuar aquí
    pass

```

![image](https://github.com/BrandonMuro2/GPT-Button/assets/80359445/5e43bac8-02de-4767-9322-f5bd57e71b22)






