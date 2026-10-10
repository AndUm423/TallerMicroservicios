# Respuestas del taller

Nombre del estudiante: _Edgar Andrés Umaña Pire_

URL de la función Serverless: _(opcional)_

---

## Preguntas de análisis — Microservicios

### 1. ¿Qué responsabilidad tiene cada microservicio?

**Respuesta:**

- **Servicio de pedidos (puerto 3001):** recibe la solicitud del cliente, verifica que se haya enviado una pizza, coordina la reserva en inventario y el pago, y devuelve la confirmación del pedido. También puede llamar al servicio de notificaciones si se activa esa opción.
- **Servicio de inventario (puerto 3002):** consulta el menú, los precios, los ingredientes y las existencias. Cuando recibe una reserva válida, descuenta una unidad del stock.
- **Servicio de pagos (puerto 3003):** simula el cobro, genera un identificador de transacción y, en esta versión del taller, siempre aprueba el pago si recibe el monto.
- **Servicio de notificaciones (puerto 3004):** debe registrar en consola y responder un aviso cuando se confirma un pedido. En el ZIP entregado esta parte aún está pendiente de implementar.
- **API Gateway (puerto 3000):** funciona como punto de entrada único y reenvía las peticiones al servicio correspondiente. En el ZIP, el middleware de registro y algunas rutas aún están pendientes.

### 2. ¿Cómo se comunican?

**Respuesta:**

Se comunican mediante peticiones HTTP con rutas tipo API REST e intercambian datos en formato JSON. Por ejemplo, el cliente envía un `POST /pedidos` al servicio de pedidos; este solicita la reserva a `POST /inventario` en el puerto 3002 y luego envía el cobro a `POST /pagos` en el puerto 3003. Si las notificaciones están activadas, también hace una petición a `POST /notificaciones` en el puerto 3004. El gateway debe recibir las peticiones del cliente en el puerto 3000 y reenviarlas al destino correspondiente.

### 3. ¿Qué ocurre cuando un servicio deja de funcionar?

**Respuesta:**

El resultado depende del servicio que falle. Si inventario no responde, pedidos captura el error y devuelve HTTP **503**, en vez de detener todo el proceso de Node.js. Si inventario responde que no hay existencias, el pedido se rechaza con HTTP **409**. Si pagos deja de responder, pedidos devuelve HTTP **503**. La notificación es opcional: según el código de pedidos, si falla el aviso el pedido puede confirmarse de todas formas. Además, el sistema no implementa una operación de compensación para devolver al inventario una unidad si la reserva se hizo y después falla el pago; en un sistema real ese caso tendría que tratarse.

### 4. ¿Qué ventajas observaste al separar el sistema?

**Respuesta:**

Cada servicio tiene una responsabilidad concreta, por lo que su código es más fácil de entender, probar y mantener. Es posible modificar inventario sin tener que mezclar esa lógica con la de pagos, y los errores de comunicación se pueden detectar y manejar por separado. La separación también permite desplegar o escalar cada componente de acuerdo con su necesidad. El taller muestra esta independencia al ejecutar los servicios en puertos diferentes y observar qué ocurre cuando uno no está disponible.

### 5. ¿Qué problemas puede generar tener varios servicios?

**Respuesta:**

Aumenta la complejidad de instalación, configuración y monitoreo porque hay que ejecutar varios procesos y mantener sus puertos y direcciones disponibles. Las llamadas de red pueden introducir demoras, timeouts y errores si un servicio está caído. También es necesario coordinar correctamente los pasos de una operación distribuida: por ejemplo, en este código se descuenta el inventario antes de cobrar y no hay una reversión automática si el pago falla. Además, los nombres de host y puertos están configurados directamente en el código, lo que limita la flexibilidad al desplegarlo en otros entornos.

### 6. ¿Qué aprendiste al crear servicio-notificaciones?

**Respuesta:**

Aprendí a definir una ruta `POST` con Express, leer `pedidoId` y `pizza` desde `req.body`, registrar el aviso en la consola y devolver una respuesta JSON. También aprendí que un microservicio se integra con los demás mediante un contrato HTTP claro y que conviene tratar las notificaciones como una tarea secundaria: de acuerdo con el servicio de pedidos, si no se logra enviar el aviso el pedido principal puede continuar. En el ZIP recibido, los pasos de notificaciones siguen sin completar, por lo que primero hay que implementar esos bloques para obtener la respuesta exitosa descrita en la guía.

---

## Preguntas de análisis — Serverless

### 7. ¿Qué es Serverless?

**Respuesta:**

Serverless es un modelo en el que el proveedor de nube administra los servidores y el entorno de ejecución. El desarrollador despliega una función y define qué debe hacer cuando recibe una petición o evento, sin tener que mantener un servidor propio escuchando continuamente. No significa que no existan servidores, sino que su administración queda principalmente a cargo del proveedor.

### 8. ¿Qué diferencia observaste frente a los servicios locales?

**Respuesta:**

Los microservicios del taller se ejecutan como procesos de Node.js que quedan escuchando en puertos específicos, como 3001, 3002 y 3003, y hay que iniciarlos y mantenerlos disponibles. La función Serverless está escrita como un manejador que una plataforma como Vercel o Netlify puede invocar cuando llega una solicitud. Sin embargo, el archivo `server-local.js` del proyecto es solo una simulación local: inicia un servidor HTTP persistente en el puerto 3005 y **no es un despliegue Serverless real**. Para probar el modelo Serverless real hay que desplegar `index.js` en una plataforma compatible.

### 9. ¿Qué significa ejecutar una función bajo demanda?

**Respuesta:**

Significa que la plataforma invoca la función cuando llega una petición o evento, por ejemplo, un `POST` que contiene el nombre de una pizza. La función procesa los datos, construye una respuesta y termina esa invocación; el desarrollador no necesita mantener manualmente un proceso de servidor escuchando en un puerto. La plataforma administra el entorno y los recursos de ejecución.

### 10. ¿Qué ventajas y limitaciones observaste?

**Respuesta:**

Entre sus ventajas están no tener que administrar directamente el servidor, poder atender invocaciones según la demanda y contar con una implementación pequeña para tareas concretas. Entre las limitaciones están la dependencia del proveedor, los posibles arranques en frío, los límites de duración y recursos, y la necesidad de diseñar adecuadamente el estado de la aplicación. En este ejemplo, la función solo devuelve el nombre de la pizza, un mensaje, la hora de ejecución y una hora estimada 30 minutos después; no consulta inventario, no procesa pagos ni guarda un pedido real. Los tiempos de respuesta deben medirse durante las pruebas, ya que no se pueden deducir con precisión leyendo únicamente el código.

---

## Preguntas finales

### 11. ¿Qué diferencia principal encontraste entre Microservicios y Serverless?

**Respuesta:**

En este taller, los microservicios son aplicaciones independientes que permanecen ejecutándose y se comunican entre sí por HTTP para completar el proceso de un pedido. Serverless es un modelo de ejecución en el que una función se invoca bajo demanda y el proveedor administra la infraestructura. Son enfoques distintos y se pueden combinar en una misma solución; no son necesariamente excluyentes.

### 12. ¿En qué situación utilizarías una arquitectura de Microservicios?

**Respuesta:**

La utilizaría en un sistema mediano o grande con responsabilidades que puedan separarse claramente, como una tienda virtual con servicios distintos para pedidos, inventario, pagos y notificaciones. Sería especialmente útil cuando los componentes necesiten evolucionar, mantenerse o escalar de manera independiente y exista capacidad para operar y monitorear esa arquitectura distribuida.

### 13. ¿En qué situación utilizarías una función Serverless?

**Respuesta:**

La utilizaría para tareas pequeñas, puntuales o activadas por eventos, como recibir un webhook, procesar una solicitud breve, generar una notificación o transformar un archivo. Es una buena alternativa cuando no se necesita mantener un proceso escuchando continuamente y la carga puede variar, siempre que la tarea encaje en los límites y condiciones del proveedor.

### 14. ¿Qué fue lo más importante que aprendiste durante el taller?

**Respuesta:**

Lo más importante fue comprender cómo dividir una aplicación en componentes con responsabilidades claras, cómo estos se comunican mediante HTTP y JSON, y cómo manejar situaciones de error como falta de stock, servicios caídos y timeouts. También aprendí que un API Gateway puede simplificar el acceso a los servicios y que una función Serverless tiene un modelo de ejecución diferente al de un servidor local que permanece activo. La prueba real de cada escenario es importante para comprobar que el comportamiento coincide con lo esperado.


---

## Comparación Microservicios vs Serverless

| Característica | Microservicios | Serverless |
|---|---|---|
| Unidad principal | Servicio o aplicación independiente con una responsabilidad concreta. | Función o manejador que responde a una petición o evento. |
| Ejemplo del taller | `servicio-pedidos`, `servicio-inventario` y `servicio-pagos`. | `serverless/funcion-pedido/index.js`. |
| ¿Quién ejecuta el servicio? | Un proceso Node.js que se inicia y administra en el entorno donde se despliega la aplicación. | La plataforma Serverless administra el entorno y ejecuta la función. |
| ¿Cuándo se ejecuta? | Normalmente permanece escuchando solicitudes mientras el proceso está activo. | Se invoca cuando llega una petición o evento; la plataforma gestiona las instancias. |
| Comunicación | Llamadas HTTP entre servicios, generalmente intercambiando JSON. | La función recibe una invocación y devuelve una respuesta; puede integrarse con otros servicios si se programa para ello. |
| Infraestructura | El equipo debe configurar y supervisar los procesos, puertos, disponibilidad y despliegues. | El proveedor administra los servidores y buena parte del escalado, aunque se deben configurar y desplegar la función. |
| ¿Qué pudo observar el estudiante? | Puertos separados, flujo entre pedidos, inventario y pagos, y errores cuando una dependencia no responde. | La respuesta incluye la pizza, `ejecutadaEn` y `entregaEstimada`. La prueba con `server-local.js` simula la función, pero no demuestra por sí sola un despliegue Serverless real. |

---

## Preguntas de análisis — API Gateway

### 15. ¿Qué ventaja tiene que el cliente no sepa que hay varios servicios distintos detrás del gateway?

**Respuesta:**

El cliente utiliza una sola dirección de entrada y no necesita conocer los puertos ni la ubicación de cada microservicio. Esto reduce el acoplamiento entre el cliente y la estructura interna: si se cambia la ubicación de un servicio o se reorganiza el sistema, se puede ajustar el gateway sin tener que cambiar necesariamente a todos los clientes. Además, el gateway ofrece un lugar central para registrar las peticiones y puede ampliarse para aplicar políticas comunes, como autenticación o límites de solicitudes. En el código del taller, el reenvío y el registro deben completarse para que ese comportamiento funcione.

---

## Observaciones de las pruebas

Anota lo que observaste en cada actividad (pedidos, sin stock, servicio caído,
timeout, cuarto microservicio, Serverless, gateway).

- **Actividad 2 (pedidos):** La respuesta exitosa debe incluir `success: true`, `pedidoId`, `pizza`, `precio`, `stockRestante` y `transaccion`. Los precios iniciales del menú son 18000 para margarita, 22000 para hawaiana, 24000 para pepperoni, 21000 para vegetariana y 23000 para pollo. El stock disminuye en una unidad por cada reserva correcta. El identificador de transacción se genera de forma aleatoria y no es fijo.
- **Actividad 3 (sin stock):** Si el stock de la pizza se cambia a cero, inventario indica que no está disponible y pedidos devuelve HTTP 409 con un mensaje de rechazo. Después de la prueba, se debe restaurar el stock si se desea continuar.
- **Actividad 4 (servicio detenido):** Si inventario no está ejecutándose, la llamada desde pedidos falla y el servicio de pedidos devuelve HTTP 503 con un mensaje que indica que no se pudo contactar a inventario. El manejo del error evita que esa excepción detenga deliberadamente el proceso completo.
- **Actividad 5 (timeout):** Si `TIMEOUT_MS` se reduce a `1`, la petición puede ser abortada antes de recibir respuesta y pedidos probablemente devuelve HTTP 503. El resultado depende del entorno y del tiempo de respuesta; después se debe restaurar el valor de 5000 ms.
- **Cuarto microservicio:** En el ZIP original, `servicio-notificaciones/index.js` todavía tiene los bloques `COMPLETAR` y responde HTTP 501. Una vez implementado como indica la guía, debe responder con `success: true` y el mensaje de notificación. Para integrarlo con pedidos, también hay que cambiar `NOTIFICACIONES_ACTIVAS` a `true` y reiniciar el servicio de pedidos.
- **Serverless (tiempos de respuesta):** La función devuelve las marcas de tiempo `ejecutadaEn` y `entregaEstimada`; esta última se calcula 30 minutos después de la primera. No se anotan milisegundos concretos porque requieren ejecutar varias peticiones y medirlas en el entorno de prueba. La ejecución mediante `server-local.js` es una simulación local, no un despliegue Serverless real.
- **Actividad 6 (pruebas a través del gateway):** El resultado esperado tras completar el gateway es que las peticiones a `localhost:3000` se reenvíen al servicio correspondiente y conserven su código de estado y respuesta, mientras la consola registra el método y la ruta. En el ZIP original, el middleware lanza un error y las rutas de inventario y pagos aún tienen bloques `COMPLETAR`, por lo que es necesario terminarlos antes de que la actividad funcione.
