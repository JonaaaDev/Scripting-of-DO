# a expremir el codigo
import pygame
import math
import os

# --- CONFIGURACIÓN ---
ANCHO = 1024
ALTO = 768
FPS = 60
VELOCIDAD = 6
CARPETA_IMAGENES = "naves_recortadas"

# --- AJUSTES DE DIRECCIÓN ---
# Si la nave gira al revés (vas arriba y mira abajo), pon esto en True.
INVERTIR_GIRO = True 

# Ajuste fino por si la imagen 0 no es perfectamente derecha.
# Prueba con 0 primero.
OFFSET_ROTACION = 0 

class Nave(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.sprites = []
        self.cargar_sprites()
        
        # Estado inicial
        self.image = self.sprites[0]
        self.rect = self.image.get_rect()
        self.pos_x = ANCHO // 2
        self.pos_y = ALTO // 2
        self.rect.center = (self.pos_x, self.pos_y)
        
        self.target_x = self.pos_x
        self.target_y = self.pos_y
        self.moviendo = False

    def cargar_sprites(self):
        # Carga las 72 imágenes
        if not os.path.exists(CARPETA_IMAGENES):
            print(f"ERROR: No existe la carpeta '{CARPETA_IMAGENES}'.")
            exit()
            
        print("Cargando sprites...")
        for i in range(72):
            nombre = f"italy{i:04d}.png"
            ruta = os.path.join(CARPETA_IMAGENES, nombre)
            try:
                img = pygame.image.load(ruta).convert_alpha()
                self.sprites.append(img)
            except FileNotFoundError:
                print(f"Falta el archivo: {nombre}")
                exit()

    def ir_a(self, x, y):
        self.target_x = x
        self.target_y = y
        self.moviendo = True

    def update(self):
        if not self.moviendo:
            return

        # 1. Distancia
        dx = self.target_x - self.pos_x
        dy = self.target_y - self.pos_y
        distancia = math.hypot(dx, dy)

        if distancia < VELOCIDAD:
            self.pos_x = self.target_x
            self.pos_y = self.target_y
            self.moviendo = False
        else:
            # 2. Movimiento
            self.pos_x += (dx / distancia) * VELOCIDAD
            self.pos_y += (dy / distancia) * VELOCIDAD

            # 3. CÁLCULO DE ÁNGULO CORREGIDO
            angulo_rad = math.atan2(dy, dx)
            angulo_grados = math.degrees(angulo_rad)
            
            # Normalizar a 0-360
            if angulo_grados < 0:
                angulo_grados += 360

            # --- CORRECCIÓN CRÍTICA ---
            # Si los sprites están ordenados Anti-Horario (Counter-Clockwise),
            # necesitamos invertir el ángulo matemático.
            if INVERTIR_GIRO:
                angulo_grados = 360 - angulo_grados

            # Calcular índice (360 grados / 72 imágenes = 5 grados por imagen)
            indice = int(angulo_grados / 5)
            
            # Aplicar offset y asegurar que esté entre 0 y 71
            indice_final = (indice + OFFSET_ROTACION) % 72
            
            self.image = self.sprites[indice_final]

        self.rect.center = (int(self.pos_x), int(self.pos_y))

# --- JUEGO ---
pygame.init()
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Nave Point & Click - Corrección de Giro")
reloj = pygame.time.Clock()

grupo = pygame.sprite.Group()
nave = Nave()
grupo.add(nave)

ejecutando = True
while ejecutando:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            ejecutando = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1:
                nave.ir_a(*pygame.mouse.get_pos())

    grupo.update()
    
    pantalla.fill((15, 15, 30)) # Fondo espacial
    
    # Dibujar línea y punto destino
    if nave.moviendo:
        pygame.draw.line(pantalla, (50, 100, 50), nave.rect.center, (nave.target_x, nave.target_y), 1)
        pygame.draw.circle(pantalla, (0, 255, 0), (int(nave.target_x), int(nave.target_y)), 3)

    grupo.draw(pantalla)
    pygame.display.flip()
    reloj.tick(FPS)

pygame.quit()
