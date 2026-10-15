# Práctico 3 — Desarrollo Seguro de Software

**Aplicaciones:** OWASP Juice Shop y OWASP crAPI.

**Entorno de trabajo:** Kali Linux, Docker y Firefox (Network / Edit and Resend).

En este práctico se realizaron cinco desafíos de OWASP Juice Shop y cinco de OWASP crAPI, utilizando aplicaciones vulnerables en un entorno de laboratorio.

Para cada desafío se explica la vulnerabilidad encontrada, su clasificación OWASP, los pasos realizados, el resultado obtenido y una posible solución.

## 1. OWASP Juice Shop

### J1 — Zero Stars

**Vulnerabilidad:** Validación insuficiente de calificaciones.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design.

**Defecto:** La interfaz permite seleccionar entre una y cinco estrellas, pero el servidor no valida correctamente ese límite.

**Explotación:**
1. Entramos a Customer Feedback y enviamos una valoración de una estrella.
2. Capturamos la solicitud `POST /api/Feedbacks/` desde Firefox Network.
3. Utilizamos Edit and Resend para modificar `rating: 1` por `rating: 0`.
4. Reenviamos la solicitud.

**Resultado:** El servidor respondió `201 Created`, aceptó cero estrellas y Juice Shop confirmó el desafío.

**Mitigación:** Validar del lado del servidor que la calificación sea un número entero entre 1 y 5.

**Evidencia:**

![J1 Zero Stars](evidencias/Captura%20de%20pantalla%202026-10-14%20191142.png)

---

### J2 — Login Admin

**Vulnerabilidad:** SQL Injection.

**Clasificación:** OWASP Top 10 2021 — A03: Injection.

**Defecto:** El sistema permite alterar la consulta SQL utilizada para iniciar sesión mediante el campo de correo electrónico.

**Explotación:**
1. Abrimos el formulario de inicio de sesión.
2. Ingresamos `admin@juice-sh.op'--` en el campo Email y una contraseña de prueba.
3. Enviamos el formulario y observamos la solicitud `POST login` en Network.

**Resultado:** Pudimos iniciar sesión como administrador sin conocer su contraseña. Juice Shop confirmó el desafío Login Admin.

**Mitigación:** Utilizar consultas SQL parametrizadas para impedir que los datos del usuario modifiquen la consulta.

**Evidencias:**

![J2 Solicitud Login Admin](evidencias/j2-login-admin-request.png)

![J2 Resultado Login Admin](evidencias/j2-login-admin-resultado.png)

---

### J4 — Admin Registration

**Vulnerabilidad:** Mass Assignment.

**Clasificación:** OWASP Top 10 2021 — A01: Broken Access Control.

**Defecto:** El servidor permite que el usuario indique su propio rol al registrarse, aunque debería asignarlo automáticamente.

**Explotación:**
1. Registramos una cuenta normal y capturamos `POST /api/Users/`.
2. Utilizamos Edit and Resend para modificar los datos del registro.
3. Cambiamos el correo y agregamos `"role":"admin"` al JSON.
4. Reenviamos la solicitud.

**Resultado:** El servidor respondió `201 Created` y Juice Shop confirmó que se registró una cuenta con privilegios de administrador.

**Mitigación:** Asignar los roles desde el servidor y no permitir que el cliente modifique campos sensibles como `role`.

**Evidencia:**

![J4 Admin Registration](evidencias/j4-admin-registration.png.png)

---

### J6 — View Basket

**Vulnerabilidad:** IDOR (Insecure Direct Object Reference).

**Clasificación:** OWASP Top 10 2021 — A01: Broken Access Control.

**Defecto:** El sistema no verifica correctamente que el carrito solicitado pertenezca al usuario autenticado.

**Explotación:**
1. Iniciamos sesión con una cuenta normal.
2. Abrimos nuestro carrito e identificamos la solicitud `GET /rest/basket/6`.
3. Con Edit and Resend cambiamos el identificador del carrito de `6` a `1`.
4. Enviamos la solicitud manteniendo nuestra sesión.

**Resultado:** El servidor respondió `200 OK` y permitió consultar un carrito ajeno. Juice Shop confirmó View Basket.

**Mitigación:** Comprobar en el servidor que el usuario tenga autorización para acceder al carrito solicitado.

**Evidencia:**

![J6 View Basket](evidencias/j6-view-basket.png.png)

---

### J7 — Christmas Special

**Vulnerabilidad:** SQL Injection y validación insuficiente de disponibilidad de productos.

**Clasificación:** OWASP Top 10 2021 — A03: Injection y A04: Insecure Design.

**Defecto:** La búsqueda permite encontrar productos retirados y el servidor permite agregarlos al carrito y comprarlos.

**Explotación:**
1. Consultamos `/rest/products/search?q='))--`.
2. Encontramos Christmas Super-Surprise-Box (2014 Edition), con ID `10` y una fecha de eliminación.
3. Capturamos `POST /api/BasketItems/` desde Firefox Network.
4. Modificamos `ProductId` a `10` y enviamos la solicitud.
5. Agregamos el producto retirado al carrito y completamos la compra.

**Resultado:** El producto se compró correctamente y Juice Shop confirmó el desafío Christmas Special.

**Mitigación:** Utilizar consultas SQL parametrizadas y comprobar que los productos estén disponibles antes de permitir su compra.

**Evidencias:**

![J7 Producto retirado](evidencias/j7-christmas-producto.png)

![J7 Compra completada](evidencias/j7-christmas-resultado.png)

---

## 2. OWASP crAPI

### C1 — Find Another User's Vehicle Location

**Vulnerabilidad:** BOLA (Broken Object Level Authorization).

**Clasificación:** OWASP API Security Top 10 2023 — API1.

**Defecto:** El servidor permite consultar la ubicación de vehículos sin verificar que pertenezcan al usuario autenticado.

**Explotación:**
1. Capturamos la solicitud utilizada para obtener la ubicación de nuestro vehículo.
2. En Community encontramos el identificador `author.vehicleId` de un vehículo de Robot.
3. Modificamos la solicitud `GET /identity/api/v2/vehicle/{id}/location`, reemplazando el identificador por el del vehículo ajeno.
4. Enviamos la solicitud utilizando nuestra cuenta.

**Resultado:** El servidor respondió `200 OK` y devolvió las coordenadas y los datos del vehículo de Robot.

**Mitigación:** Comprobar en el servidor que el usuario tenga autorización para consultar el vehículo solicitado.

**Evidencia:**

![C1 Ubicación de vehículo ajeno](evidencias/c1-vehicle-location.png)

---

### C3 — Get an Item for Free

**Vulnerabilidad:** Validación insuficiente de cantidades y falla en la lógica de compra.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design.

**Defecto:** El sistema acepta cantidades negativas y calcula importes que no deberían permitirse.

**Explotación:**
1. Compramos un Seat, cuyo precio era $10.
2. Capturamos la solicitud `POST /workshop/api/shop/orders`.
3. Con Edit and Resend enviamos:

`{"product_id":1,"quantity":-1}`

**Resultado:** El servidor respondió `200 OK` y registró un pedido con cantidad `-1` y total `-$10`.

Se utilizó una compra con cantidad negativa como variante del desafío. Esto demuestra el fallo en la lógica de cobro, aunque no se realizó una devolución de producto como menciona la consigna.

**Mitigación:** Validar que las cantidades sean números enteros positivos y rechazar valores negativos.

**Evidencia:**

![C3 Pedido negativo](evidencias/c3-pedido-negativo.png)

---

### C4 — Increase Balance by $1,000 or More

**Vulnerabilidad:** Manipulación del saldo mediante cantidades negativas.

**Clasificación:** OWASP Top 10 2021 — A04: Insecure Design.

**Defecto:** El sistema no valida las cantidades antes de calcular el importe y actualizar el saldo.

**Explotación:**
1. Comprobamos que el saldo inicial era de $100.
2. Realizamos una compra normal de Seat por $10.
3. Capturamos `POST /workshop/api/shop/orders` y reenviamos:

`{"product_id":1,"quantity":-101}`

4. Comprobamos el saldo final.

**Resultado:** La API aceptó la solicitud y el saldo pasó de $100 a $1.100.

El cálculo fue:

`$100 - $10 + $1.010 = $1.100`

La operación manipulada generó un crédito de $1.010, cumpliendo el objetivo de aumentar el saldo al menos $1.000.

**Mitigación:** Rechazar cantidades negativas y verificar los importes antes de modificar el saldo.

**Evidencias:**

![C4 Saldo inicial](evidencias/c4-saldo-inicial.png)

![C4 Saldo final](evidencias/c4-saldo-final.png)

---

### C5 — Server-Side Request Forgery (SSRF)

**Vulnerabilidad:** SSRF (Server-Side Request Forgery).

**Clasificación:** OWASP API Security Top 10 2023 — API7.

**Defecto:** El sistema permite que el cliente indique a qué dirección debe enviar una solicitud el servidor.

**Explotación:**
1. Entramos a Contact Mechanic.
2. Capturamos la solicitud `POST contact_mechanic`.
3. Utilizamos Edit and Resend para modificar el parámetro `mechanic_api`.
4. Reemplazamos su valor por `https://www.google.com`.
5. Enviamos la solicitud modificada.

**Resultado:** El servidor respondió `200 OK` y devolvió contenido HTML de Google dentro de `response_from_mechanic_api`.

Esto demuestra que el servidor realizó una solicitud a una dirección elegida por nosotros en lugar de utilizar únicamente el servicio del mecánico.

**Mitigación:** Permitir solamente destinos autorizados y validar las direcciones del lado del servidor.

**Evidencia:**

![C5 SSRF Google](evidencias/c5-ssrf-google.png)

---

### C6 — NoSQL Injection

**Vulnerabilidad:** NoSQL Injection.

**Clasificación:** OWASP API Security Top 10 2019 — API8: Injection.

**Defecto:** El servidor acepta operadores de consulta NoSQL en un campo que debería recibir solamente texto.

**Explotación:**
1. En Shop → Add Coupons, ingresamos un código inválido de prueba.
2. Capturamos la solicitud `POST validate-coupon`.
3. Mediante Edit and Resend enviamos:

`{"coupon_code":{"$ne":null}}`

4. La API devolvió un código de cupón que no habíamos proporcionado.
5. Ingresamos ese código en Add Coupons para comprobar si funcionaba.

**Resultado:** Obtuvimos el cupón `TRAC075`, por un importe de $75. La aplicación confirmó Coupon applied y el saldo aumentó de $1.100 a $1.175.

**Mitigación:** Validar que `coupon_code` sea una cadena de texto y rechazar objetos que contengan operadores NoSQL.

**Evidencias:**

![C6 Cupón obtenido](evidencias/c6-nosql-cupon.png)

![C6 Cupón aplicado](evidencias/c6-cupon-aplicado.png)

---

## Conclusión

Las pruebas realizadas permitieron identificar vulnerabilidades relacionadas con inyección SQL y NoSQL, controles de acceso insuficientes, falta de validaciones y errores en la lógica de negocio.

En cada ejercicio se documentó el procedimiento, el resultado obtenido y una medida de mitigación.

Todas las pruebas se realizaron en aplicaciones vulnerables de laboratorio. Las capturas correspondientes se encuentran en la carpeta `evidencias/`.
