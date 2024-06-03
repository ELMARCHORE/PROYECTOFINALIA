# Proyecto Inteligencia Artificial
# PREDICCION DEL CLIMA

- **Nombre:**  Luis Elmar Chore Tomicha
- **Universidad:** Universidad Privada Domingo Savio
- **Carrera:** Ingeniería de Sistemas
- **Facebook:** [elmander.maronejr](https://www.facebook.com/sergiomaronejr?mibextid=ZbWKwL)

## Introducción
Este proyecto tiene como objetivo desarrollar un sistema de visualización y síntesis de información meteorológica utilizando la API de OpenWeatherMap, Pygame para la interfaz gráfica y pyttsx3 para la síntesis de voz.


## Objetivo
Crear una aplicación que permita a los usuarios obtener información meteorológica detallada de una ciudad específica, mostrar dicha información en una interfaz gráfica y permitir la lectura en voz alta de los datos obtenidos.

## Marco Teórico

### Pygame
Pygame es una biblioteca de Python que permite el desarrollo de videojuegos y aplicaciones multimedia. Proporciona módulos para gestionar gráficos, sonidos y eventos.

### pyttsx3
pyttsx3 es un módulo de Python que permite la conversión de texto a voz. A diferencia de otros módulos, pyttsx3 funciona sin conexión a Internet.

### OpenWeatherMap
OpenWeatherMap proporciona datos meteorológicos globales mediante su API. Ofrece información detallada del clima, como temperatura, humedad, presión y velocidad del viento.


## Código Fuente y Procedimientos de Instalación
- **Instalación de Dependencias**
```bash
pip install pygame pyttsx3 requests

```
```bash
import requests
import pygame
import os
import pyttsx3
import threading

# Introduce tu clave de API de OpenWeatherMap aquí
api_key = '257d0f9ae1c9565da6a40682a7deb497'

# Inicializar Pygame
pygame.init()
font = pygame.font.SysFont('Arial Black', 12)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Inicializar el motor de síntesis de voz
engine = pyttsx3.init()

# Función para mostrar texto en pantalla con tamaño de fuente ajustable
def draw_text(text, font, color, surface, x, y, max_width):
    words = text.split()
    lines = []
    while words:
        line_words = []
        while words and font.size(' '.join(line_words + [words[0]]))[0] <= max_width:
            line_words.append(words.pop(0))
        lines.append(' '.join(line_words))
    
    y_offset = y
    for line in lines:
        textobj = font.render(line, True, color)
        textrect = textobj.get_rect()
        textrect.topleft = (x, y_offset)
        surface.blit(textobj, textrect)
        y_offset += font.get_linesize()

# Función para decir el texto mediante audio en un hilo separado
def speak_text(text):
    def run():
        engine.say(text)
        engine.runAndWait()

    t = threading.Thread(target=run)
    t.start()

# Función para detener la lectura del clima
def stop_speaking():
    engine.stop()

# Función para descargar iconos de clima
def download_weather_icon(icon_id):
    url = f"http://openweathermap.org/img/wn/{icon_id}@2x.png"
    response = requests.get(url)
    if response.status_code == 200:
        file_path = os.path.join('icons', f"{icon_id}.png")
        with open(file_path, 'wb') as f:
            f.write(response.content)
        return file_path
    else:
        print("Error al descargar el icono del clima.")
        return None

# Función para obtener datos del clima
def get_weather_data(city):
    url = f"http://api.openweathermap.org/data/2.5/forecast?q={city}&appid={api_key}&units=metric&lang=es"
    res = requests.get(url)
    if res.status_code == 200:
        return res.json()
    else:
        print("Error al obtener los datos del clima.")
        return None

# Función para mostrar la ventana de entrada
def input_window():
    input_active = True
    input_text = ''
    screen = pygame.display.set_mode((400, 200))
    pygame.display.set_caption("Ingrese el nombre de la ciudad")
    while input_active:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:
                    input_active = False
                    return input_text  # Devolver el texto ingresado
                elif event.key == pygame.K_BACKSPACE:
                    input_text = input_text[:-1]
                else:
                    input_text += event.unicode
        
        screen.fill(WHITE)
        draw_text("Ingrese el nombre de la ciudad:", font, BLACK, screen, 20, 50, 360)
        draw_text(input_text, font, BLACK, screen, 20, 100, 360)
        pygame.display.flip()
    
    return input_text

# Función para mostrar la ventana del clima
def weather_window(city):
    weather_data = get_weather_data(city)
    
    if weather_data is None:
        print("Error al obtener los datos del clima.")
        return

    city_name = weather_data['city']['name']
    country = weather_data['city']['country']
    lat = weather_data['city']['coord']['lat']
    lon = weather_data['city']['coord']['lon']
    forecasts = weather_data['list'][:40]  # Obtener 40 periodos de 3 horas (5 días)
    weather_infos = []

    # Crear la carpeta de iconos si no existe
    if not os.path.exists('icons'):
        os.makedirs('icons')

    for forecast in forecasts:
        dt_txt = forecast['dt_txt']
        date, time_txt = dt_txt.split(' ')
        temp = forecast['main']['temp']
        description = forecast['weather'][0]['description']
        wind_speed = forecast['wind']['speed']
        humidity = forecast['main']['humidity']
        pressure = forecast['main']['pressure']
        icon_id = forecast['weather'][0]['icon']

        icon_path = download_weather_icon(icon_id)

        weather_infos.append({
            'date': date,
            'time': time_txt,
            'temp': temp,
            'description': description,
            'wind_speed': wind_speed,
            'humidity': humidity,
            'pressure': pressure,
            'icon_path': icon_path
        })

    WIDTH, HEIGHT = 1024, 768  # Asignar valores a WIDTH y HEIGHT aquí

    screen = pygame.display.set_mode((WIDTH, HEIGHT), pygame.RESIZABLE)
    pygame.display.set_caption(f"Weather App - {city_name}, {country}")

    scroll_x = 0
    scroll_y = 0
    scroll_speed = 20
    running = True

    button_speak_rect = pygame.Rect(WIDTH - 300, HEIGHT - 50, 130, 30)
    button_stop_rect = pygame.Rect(WIDTH - 150, HEIGHT - 50, 130, 30)
    button_color = (0, 128, 255)
    button_speak_text = font.render("Leer Clima", True, WHITE)
    button_stop_text = font.render("Detener", True, WHITE)

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.VIDEORESIZE:
                WIDTH, HEIGHT = event.w, event.h
                screen = pygame.display.set_mode((WIDTH, HEIGHT), pygame.RESIZABLE)
                button_speak_rect.topleft = (WIDTH - 300, HEIGHT - 50)
                button_stop_rect.topleft = (WIDTH - 150, HEIGHT - 50)
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    scroll_y = max(scroll_y - scroll_speed, 0)
                elif event.key == pygame.K_DOWN:
                    scroll_y = min(scroll_y + scroll_speed, (len(weather_infos) // 4) * 160 + 40 - HEIGHT)
                elif event.key == pygame.K_LEFT:
                    scroll_x = max(scroll_x - scroll_speed, 0)
                elif event.key == pygame.K_RIGHT:
                    scroll_x = min(scroll_x + scroll_speed, (len(weather_infos) // 4) * 240 + 40 - WIDTH)
            elif event.type == pygame.MOUSEBUTTONDOWN:
                if button_speak_rect.collidepoint(event.pos):
                    for info in weather_infos:
                        weather_text = (
                            f"Fecha: {info['date']}, Hora: {info['time']}, "
                            f"Temperatura: {info['temp']} grados Celsius, "
                            f"Descripción: {info['description']}, "
                            f"Viento: {info['wind_speed']} metros por segundo, "
                            f"Humedad: {info['humidity']} por ciento, "
                            f"Presión: {info['pressure']} hPa."
                        )
                        speak_text(weather_text)
                elif button_stop_rect.collidepoint(event.pos):
                    stop_speaking()
                else:
                    # Verificar si se hizo clic en algún cuadro de información del clima
                    x_offset = 20 - scroll_x
                    y_offset = 20 - scroll_y
                    cell_width = 240
                    cell_height = 200
                    margin = 10
                    for info in weather_infos:
                        if x_offset <= event.pos[0] <= x_offset + cell_width and y_offset <= event.pos[1] <= y_offset + cell_height:
                            weather_text = (
                                f"Fecha: {info['date']}, Hora: {info['time']}, "
                                f"Temperatura: {info['temp']} grados Celsius, "
                                f"Descripción: {info['description']}, "
                                f"Viento: {info['wind_speed']} metros por segundo, "
                                f"Humedad: {info['humidity']} por ciento, "
                                f"Presión: {info['pressure']} hPa."
                            )
                            speak_text(weather_text)
                            break  # Romper el bucle una vez que se encuentra el cuadro clicado
                        x_offset += cell_width + margin
                        if x_offset + cell_width > WIDTH:
                            x_offset = 20
                            y_offset += cell_height + margin

        screen.fill(WHITE)

        # Mostrar los datos en una cuadrícula
        x_offset = 20 - scroll_x
        y_offset = 20 - scroll_y
        cell_width = 240
        cell_height = 200
        margin = 10

        for info in weather_infos:
            pygame.draw.rect(screen, BLACK, (x_offset, y_offset, cell_width, cell_height), 2) 
            # Dibujar el marco

            if info['icon_path']:
                icon = pygame.image.load(info['icon_path'])
                icon = pygame.transform.scale(icon, (40, 40))
                screen.blit(icon, (x_offset + 5, y_offset + 5))

            draw_text(f"Fecha: {info['date']}", font, BLACK, screen, x_offset + 50, y_offset + 5, cell_width - 60)
            draw_text(f"Hora: {info['time']}", font, BLACK, screen, x_offset + 50, y_offset + 25, cell_width - 60)
            draw_text(f"Temperatura: {info['temp']}°C", font, BLACK, screen, x_offset + 50, y_offset + 45, cell_width - 60)
            draw_text(f"Descripción: {info['description']}", font, BLACK, screen, x_offset + 50, y_offset + 65, cell_width - 60)
            draw_text(f"Viento: {info['wind_speed']} m/s", font, BLACK, screen, x_offset + 50, y_offset + 85, cell_width - 60)
            draw_text(f"Humedad: {info['humidity']}%", font, BLACK, screen, x_offset + 50, y_offset + 105, cell_width - 60)
            draw_text(f"Presión: {info['pressure']} hPa", font, BLACK, screen, x_offset + 50, y_offset + 125, cell_width - 60)

            x_offset += cell_width + margin
            if x_offset + cell_width > WIDTH:
                x_offset = 20
                y_offset += cell_height + margin

        # Dibujar los botones
        pygame.draw.rect(screen, button_color, button_speak_rect)
        screen.blit(button_speak_text, (button_speak_rect.x + 10, button_speak_rect.y + 5))

        pygame.draw.rect(screen, button_color, button_stop_rect)
        screen.blit(button_stop_text, (button_stop_rect.x + 20, button_stop_rect.y + 5))

        pygame.display.flip()

# Ejecutar el programa
city = input_window()
weather_window(city)
pygame.quit()


```
## 🖥️ Modelado o Sistematización

El sistema se basa en las bibliotecas Pygame y pyttsx3 para crear una interfaz de usuario y proporcionar síntesis de voz, respectivamente. La API de OpenWeatherMap se utiliza para obtener datos meteorológicos actualizados.

## 📊 Conclusiones
El sistema desarrollado permite a los usuarios obtener y visualizar información meteorológica detallada de manera interactiva. Además, la síntesis de voz mejora la accesibilidad del sistema, haciendo que la información sea accesible para personas con discapacidades visuales.

## 📚 Bibliografía

- **OpenWeatherMap. (2024).** OpenWeatherMap API. Recuperado de https://openweathermap.org/
- **Pygame. (2024).** Pygame Documentation. Recuperado de https://www.pygame.org/docs/
- **pyttsx3. (2024).** pyttsx3 Documentation. Recuperado de https://pyttsx3.readthedocs.io/
- **Codigo badse e informacion adicional**  . Recuperado de https://tecnonovax.wordpress.com/saber-el-clima-con-python/
## 📁 Anexos
- **Código Fuente:** [GitHub]()
- **Documentación de OpenAI:** [OpenAIDocs](https://platform.openai.com/docs/introduction)