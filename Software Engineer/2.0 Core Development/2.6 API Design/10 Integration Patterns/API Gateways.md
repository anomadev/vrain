## Definición
Un `API Gateway` es una herramienta de gestión de tráfico que actúa como un punto de entrada único (punto de contacto centralizado) entre un cliente y un grupo de microservicios o servicios `backend`. En lugar de que un cliente llame a diez servicios diferentes para cargar una pantalla, realiza una única llamada al `Gateway`, y este se encarga de enrutar, proteger y transformar las peticiones según sea necesario.

### Funciones Principales
Basado en las perspectivas de `Red Hat` e `IBM`, un `API Gateway` no solo mueve paquetes de datos; es el "cerebro" de la infraestructura de comunicación.
- **Enrutamiento de Peticiones (`Routing`)**: Dirige la solicitud al microservicio correcto basándose en la `URL` o los `headers`.
- **Seguridad y Autenticación**: Centraliza la validación de `tokens` (`JWT`, `OAuth`) y `API Keys`. Si el usuario no está autorizado, la petición ni siquiera llega a tus servidores internos.
- **`Rate Limiting` y `Throttling`**: Protege tus servicios de abusos o picos de tráfico limitando cuántas peticiones puede hacer un cliente por segundo.
- **Traducción de Protocolos**: Puede recibir una petición `REST/JSON` del cliente y transformarla internamente a `gRPC` o `AMQP` para hablar con tus microservicios. 
- **`Caching**:` Almacena respuestas comunes para servirlas instantáneamente sin tocar le `backend`, mejorando la latencia.
- **Monitoreo y `Logging`**: Registra métricas de uso y errores en un solo lugar, facilitando la observabilidad de todo el sistema. 
### Beneficios de Implementación
| Beneficio               | Descripción                                                                                |
| :---------------------- | ------------------------------------------------------------------------------------------ |
| Desacoplamiento         | Los clientes no necesitan saber cuántos microservicios tienes ni dónde están.              |
| Seguridad Centralizada  | No tienes que implementar lógica de autenticación en cada uno de tus 20 microservicios.    |
| Simplicidad del Cliente | El `frontend` hace menos llamadas de red, ahorrando batería y datos.                       |
| Gobernanza              | Permite aplica políticas de uso globales (ej. "Nadie puede llamar a la `API` sin `HTTPS`). |
### Diferencia Crítica: `API Gateway` vs. `Load Balancer`
Aunque a veces se confunden, su propósito es distinto:
- **`Load Balancer`**: Su misión es la **disponibilidad**. Reparte la carga entre copias idénticas de un mismo servicio.
- **`API Gateway`**: Su misión es la **gestión y orquestación**. Decide a qué servicio distinto (`A`, `B`, `C`) debe ir la petición y bajo qué reglas de negocio.
### Herramientas Populares en el Mercado
- **`Kong`**: Altamente extensible, basado en `Nginx` y muy popular en la comunidad `open-source`
- **`Tyk`**: Escrito en `Go`, enfocado en el rendimiento y la facilidad de uso.
- **`AWS API Gateway`**: Servicio gestionado ideal para arquitecturas Serverless (`Lambda`).
- **`KrakenD`**: Un `Gateway` ultra rápido enfocado en el rendimiento puro, escrito en `Go`.
- **`Apigee (Google Cloud)`**: Solución empresarial de alto nivel con analíticas avanzadas.

## Analogía
Imagina el **`API Gateway` como Conserje de un Hotel de Lujo**:
- Los clientes (`frontend`) no entra a la cocina a pedir comida, ni van a la lavandería a pedir toallas.
- Hablan con el Conserje (`Gateway`).
- El conserje verifica si tienen reserva (`Authentication`), anota el pedido, llama a la cocina o a limpieza (`Routing`), y si el cliente pide demasiadas cosas en un minuto, le pide que espere (`Rate Limiting`).
- Al final, el conserje le entrega todo al cliente en una sola bandeja.