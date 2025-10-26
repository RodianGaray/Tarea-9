# Repositorio: Juegos-Digitales

Este repositorio contiene la estructura propuesta y el código base para crear un repositorio en GitHub que reúna **dos proyectos** (Brazo Robótico con PyBullet y Pista de Carreras con Tkinter) más un contenedor para los juegos (`juegos_digitales`). Incluye Dockerfiles listos para construir imágenes y un README con instrucciones.

---

## 1. Construir imágenes Docker (ejemplo):

```bash
docker build -t brazo_robotico:latest ./pybullet_arm
docker build -t pista_carreras:latest ./pista_carreras
docker build -t juegos_digitales:latest ./juegos_digitales
```

## 2. Ejecutar contenedor (PyBullet — sin GUI o con X forwarding):

```bash
# Sin GUI (headless)
docker run --rm brazo_robotico:latest

# Con GUI (Linux):
# xhost +local:docker
# docker run --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix brazo_robotico:latest
```

## 3. Ejecutar contenedor (Tkinter — necesita DISPLAY):

```bash
# Linux:
xhost +local:docker
docker run --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix pista_carreras:latest
```

---

## .gitignore

```
__pycache__/
*.pyc
*.pyo
.env
.vscode/
.DS_Store
docker-compose.override.yml
```

---

## Archivo: pybullet_arm/robotic_arm.py

```python
# robotic_arm.py - ejemplo mínimo con pybullet
import pybullet as p
import pybullet_data
import time

def main():
    physicsClient = p.connect(p.GUI)
    p.setAdditionalSearchPath(pybullet_data.getDataPath())
    p.setGravity(0,0,-9.81)
    plane = p.loadURDF("plane.urdf")
    robot = p.loadURDF("kuka_iiwa/model.urdf", basePosition=[0,0,0])

    for i in range(10000):
        t = i*0.01
        # un movimiento simple de ejemplo: mover la primera articulacion con una señal senoidal
        target = 0.5 * p.sin(t)
        p.setJointMotorControl2(robot, 0, p.POSITION_CONTROL, targetPosition=target)
        p.stepSimulation()
        time.sleep(1./240.)

    p.disconnect()

if __name__ == '__main__':
    main()
```

**Nota:** Este ejemplo usa un URDF disponible en pybullet_data (`kuka_iiwa`). Ajustar a tu propio URDF si lo tienes.

---

## Archivo: pybullet_arm/Dockerfile

```dockerfile
FROM python:3.11-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libgl1-mesa-glx \
    libosmesa6-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements-pybullet.txt ./
RUN pip install --no-cache-dir -r requirements-pybullet.txt
COPY robotic_arm.py ./

CMD ["python", "robotic_arm.py"]
```

Y archivo `pybullet_arm/requirements-pybullet.txt`:

```
pybullet
numpy
```

---

## Archivo: pista_carreras/pista.py

```python
# pista.py - ejemplo mínimo de una pista con Tkinter
import tkinter as tk

class PistaApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title('Pista de Carreras - Demo')
        self.geometry('800x600')
        self.canvas = tk.Canvas(self, width=800, height=600)
        self.canvas.pack()
        self.draw_track()

    def draw_track(self):
        # pista simple: rectángulo con curvas simuladas
        self.canvas.create_rectangle(50,50,750,550, outline='black', width=4)
        # cinco curvas (ejemplo de curvas con arcs)
        self.canvas.create_arc(100,100,300,300, start=0, extent=180, style='arc', width=3)
        self.canvas.create_arc(250,200,450,400, start=180, extent=180, style='arc', width=3)
        self.canvas.create_arc(400,100,600,300, start=0, extent=180, style='arc', width=3)
        self.canvas.create_arc(150,300,350,500, start=180, extent=180, style='arc', width=3)
        self.canvas.create_arc(500,300,700,500, start=180, extent=180, style='arc', width=3)

if __name__ == '__main__':
    app = PistaApp()
    app.mainloop()
```

---

## Archivo: pista_carreras/Dockerfile

```dockerfile
FROM python:3.11-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    tk \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements-tk.txt ./
RUN pip install --no-cache-dir -r requirements-tk.txt
COPY pista.py ./

CMD ["python", "pista.py"]
```

Archivo `pista_carreras/requirements-tk.txt`:

```
# Solo dependencias python adicionales si las necesitas
```

**Nota:** Para ejecutar la GUI de Tkinter desde Docker en Linux debes compartir el socket X: `-e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix`.

---

## Archivo: juegos_digitales/JuegoDef.py

```python
# JuegoDef.py - estructura base para menú y selección de opciones
import tkinter as tk

class MenuJuego(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title('Juegos Digitales - Menu')
        self.geometry('600x400')
        tk.Label(self, text='Seleccione el juego:').pack(pady=10)
        tk.Button(self, text='Brazo Robotico (Sim)', command=self.open_brazo).pack(fill='x', padx=60, pady=5)
        tk.Button(self, text='Pista de Carreras', command=self.open_pista).pack(fill='x', padx=60, pady=5)

    def open_brazo(self):
        print('Aquí se lanzaría el contenedor o la simulación del brazo')

    def open_pista(self):
        print('Aquí se lanzaría la pista')

if __name__ == '__main__':
    app = MenuJuego()
    app.mainloop()
```

---

## Archivo: juegos_digitales/Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY JuegoDef.py ./
RUN pip install --no-cache-dir tkinter || true
CMD ["python","JuegoDef.py"]
```

---
## Capturas

(Agrega tus imágenes en `docs/` o en la sección de la wiki del repo)

![Tare-5](1.jpg)
![Tare-5](1.jpg)
![Tare-5](1.jpg)
![Tare-5](1.jpg)
![Tare-5](1.jpg)
![Tare-5](1.jpg)


---
