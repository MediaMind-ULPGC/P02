# Práctica 2: Transformaciones geométricas.

Integrantes:
- Gerardo León Quintana
- Susana Suárez Mendoza

# Instrucciines de uso OPTATIVO

## 1a. Desarrollar una aplicación que lleve a cabo transformaciones de la imagen en tiempo real a través de una interfaz basada en trackbars o equivalente.
Este programa permite realizar transformaciones en tiempo real sobre una imagen mediante una interfaz que utiliza barras deslizantes (trackbars). Las transformaciones disponibles incluyen traslaciones, rotaciones y escalados, tanto uniformes como no uniformes. La interfaz es intuitiva y permite al usuario ajustar parámetros dinámicamente para visualizar los cambios en la imagen al instante.
**Flujo del programa:**
1. **Cargar la imagen**
```python
img = cv.imread('./images/img_cuadro.jpg', cv.IMREAD_COLOR)
if img is None:
    raise FileNotFoundError("La imagen no se encontró en la ruta especificada.")
```
2. **Configuración de la interfaz**: Se crean varias barras deslizantes que permiten ajustar los parámetros de las transformaciones:
- Traslación: Magnitud de la traslación en los ejes X e Y.
- Rotación: Centro de giro y ángulo de rotación.
- Escalado: Factores de escala, tanto uniformes como no uniformes.
```python
cv.createTrackbar('X', 'image', 0, width, nothing)
cv.createTrackbar('Y', 'image', 0, height, nothing)
cv.createTrackbar('Angulo', 'image', 0, 360, nothing)
cv.createTrackbar('Escala', 'image', 50, 100, nothing)
cv.createTrackbar('Escala X', 'image', 50, 100, nothing)
cv.createTrackbar('Escala Y', 'image', 50, 100, nothing)
```
3. **Visualización de la imagen**: La imagen original se muestra en una ventana, y los cambios se aplican en tiempo real. Se alterna entre diferentes modos de transformación (traslación, rotación, escalado) al presionar la tecla 'm'.
```python
while True:
    key = cv.waitKey(1) & 0xFF
    if key == ord('m'):
      ...
```
4. **Aplicación de transformaciones**: Dependiendo del modo seleccionado, se aplican las transformaciones correspondientes a la imagen.
- **Traslaciones**: La transformación de traslación permite desplazar la imagen en las direcciones de los ejes X e Y. El usuario tiene la capacidad de especificar la magnitud de la traslación mediante el uso de `trackbars`, los cuales ajustan la posición de la imagen. Esta operación se realiza aplicando la siguiente fórmula, donde OpenCV efectúa la transformación de manera directa:

<p align="center">
<img src="images/traslacion.svg" width = "250" >
</p>

```python
while True:
  ...
  if mode == 'Traslación':
        T = np.float32([[1, 0, tx], [0, 1, ty]])
        img_display = cv.warpAffine(img_display, T, size)
        texto = f'Modo: Traslacion'
  ...
```
- **Rotaciones**: La transformación de rotación permite rotar una imagen alrededor de un centro de giro, el cual se determina al pulsar el botón izquierdo del ratón. Los grados de rotación se pueden ajustar mediante un `trackbar`. Esta operación se realiza aplicando la siguiente fórmula, donde OpenCV lleva a cabo la transformación de manera directa:

<p align="center">
<img src="images/rotacion.svg" width = "300" >
</p>

```python
def get_center(event, x, y, flags, param):
    global center, show_text, text_start_time
    if event == cv.EVENT_LBUTTONDOWN:
        center = (x, y)
    elif event == cv.EVENT_LBUTTONUP:
        return center
cv.setMouseCallback('image', get_center)
while True:
  ...
  elif mode == 'Rotación':
        R = cv.getRotationMatrix2D(center, angle, 1)
        img_display = cv.warpAffine(img_display, R, size)
        texto = f'Modo: Rotacion'
  ...
```
- **Escalados uniformes**: Los escalados uniformes son transformaciones que modifican el tamaño de una imagen de manera proporcional en todas las direcciones. Para llevar a cabo esta transformación, se requieren un único factor de escalado, `escala`, que son ajustados mediante un `trackbar`. Es importante destacar que el `trackbar` se mueven en un rango de 0 a 100, lo que se traduce en un escalado negativo entre 0 y 50 y un escalado positivo entre 50 y 100. La operación se realiza utilizando la siguiente fórmula:

<p align="center">
<img src="images/escaladouniforme.svg" width = "250" >
</p>

```python
cv.createTrackbar('Escala', 'image', 50, 100, nothing)
while True:
  ...
  elif mode == 'Escalado Uniforme':
        scale_factor = escala / 50.0  
        S = cv.getRotationMatrix2D(center, 0, scale_factor)
        img_display = cv.warpAffine(img_display, S, size)
        texto = f'Modo: Escalado Uniforme'
  ...
```
- **Escalados no uniformes**: 
Los escalados no uniformes son transformaciones que modifican el tamaño de una imagen de manera diferente en las direcciones de los ejes X e Y. A diferencia del escalado uniforme, que mantiene la relación de aspecto de la imagen, el escalado no uniforme puede distorsionar la imagen al aplicar factores de escalado distintos para cada dirección.

<p align="center">
<img src="images/escaladonouniforme.svg" width = "250" >
</p>

Donde:
- $S_x$: Factor de escalado en el eje X.
- $S_y$: Factor de escalado en el eje Y.
- $t_x$ y $t_y$ son los desplazamientos en las direcciones X e Y,  que se calculan con la fórmula $(1- S_x) ^* center[0]$ y $(1- S_y) ^* center[1]$ respectivamente. Esto centra el escalado alrededor de un punto específico, dado por `center`, que es un par de coordenadas (x, y) que representa el punto de origen para el escalado.

```python
while True:
  ...
  elif mode == 'Escalado No Uniforme':
          scale_x_factor = escala_x / 50.0  
          scale_y_factor = escala_y / 50.0
  
          S = np.float32([[scale_x_factor, 0, (1 - scale_x_factor) * center[0]],
                          [0, scale_y_factor, (1 - scale_y_factor) * center[1]]])
          img_display = cv.warpAffine(img_display, S, size)
          texto = f'Modo: Escalado No Uniforme'
```
