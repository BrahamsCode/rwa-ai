# 🎤 Sistema de Pedidos por Voz para Restaurantes

Sistema completo de pedidos por voz usando n8n + Web Speech API / VAPI con integración a PostgreSQL.

## 🚀 Opción 1: Demo Rápido (5 minutos)

### Archivo: `voice-order-simple.html`

**Características:**
- ✅ Reconocimiento de voz en español
- ✅ Menú configurable con precios
- ✅ Detección automática de items y cantidades
- ✅ Cálculo automático de totales
- ✅ Interfaz responsive y moderna
- ✅ Compatible con Chrome/Edge (Web Speech API)

**Uso:**
1. Abre `voice-order-simple.html` en Chrome o Edge
2. Presiona el micrófono 🎤
3. Di tu pedido: "Quiero una pizza margherita, dos coca-colas y una ensalada césar para la mesa 5"
4. El sistema procesará automáticamente los items
5. Confirma el pedido

**Limitaciones:**
- Solo funciona en Chrome/Edge/Safari
- Requiere conexión a internet
- Solo español (puedes cambiar `recognition.lang`)

---

## 🔥 Opción 2: Sistema Completo con n8n (Producción)

### Requisitos:
- Docker
- PostgreSQL
- Cuenta en VAPI (gratis) o ElevenLabs
- (Opcional) Telegram Bot para notificaciones

### 1. Instalar n8n

```bash
# Con Docker
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e WEBHOOK_URL="http://localhost:5678/" \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Acceder a: http://localhost:5678
```

### 2. Configurar Base de Datos PostgreSQL

```sql
-- Crear base de datos
CREATE DATABASE restaurant_voice;

-- Conectar a la base de datos
\c restaurant_voice;

-- Tabla de pedidos
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    items TEXT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    table_number VARCHAR(20),
    status VARCHAR(50) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabla de reservas (opcional)
CREATE TABLE bookings (
    booking_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    phone_number VARCHAR(20),
    party_size INTEGER NOT NULL,
    booking_time TIMESTAMP NOT NULL,
    status VARCHAR(50) DEFAULT 'confirmed',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabla de información del restaurante
CREATE TABLE restaurant_info (
    info_key VARCHAR(50) PRIMARY KEY,
    info_value TEXT NOT NULL,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insertar información básica
INSERT INTO restaurant_info (info_key, info_value) VALUES 
('hours', 'Lunes a Viernes: 12:00 PM - 10:00 PM, Sábado-Domingo: 11:00 AM - 11:00 PM'),
('menu', 'Pizza Margherita $12, Pizza Pepperoni $14, Pasta Carbonara $15, Ensalada César $8, Coca-Cola $3, Sprite $3'),
('location', 'Calle Principal 123, Ciudad, CP 12345'),
('phone', '+1234567890'),
('delivery_zones', 'Zona Centro, Zona Norte, Zona Sur');

-- Índices para mejor rendimiento
CREATE INDEX idx_orders_created ON orders(created_at DESC);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_bookings_time ON bookings(booking_time);
```

### 3. Importar Workflow en n8n

1. En n8n, ve a **Workflows → Import from File**
2. Selecciona `n8n-voice-order-workflow.json`
3. Configura las credenciales:
   - **PostgreSQL**: Agrega tu conexión a la base de datos
   - **Telegram** (opcional): Crea un bot con @BotFather
   - **ElevenLabs** (opcional): Registra en https://elevenlabs.io

### 4. Configurar VAPI (Recomendado)

**Opción A: VAPI (Más fácil para voz telefónica)**

1. Registrarse en https://vapi.ai (Plan gratis: 10 min/mes)
2. Crear un nuevo Assistant con este prompt:

```
Eres un asistente de restaurante en español. Tu nombre es María.

FUNCIONES:
1. Tomar pedidos de comida
2. Hacer reservas de mesa
3. Responder preguntas sobre el menú y horarios

MENÚ:
- Pizza Margherita: $12
- Pizza Pepperoni: $14
- Pasta Carbonara: $15
- Ensalada César: $8
- Coca-Cola: $3

REGLAS:
- Siempre confirma el pedido antes de enviarlo
- Pregunta el número de mesa
- Sé amable y profesional
- Si no entiendes, pide que repitan

EJEMPLO DE CONVERSACIÓN:
Cliente: "Quiero una pizza margherita"
Tú: "¡Perfecto! Una pizza margherita por $12. ¿Para qué mesa?"
Cliente: "Mesa 5"
Tú: "Excelente. ¿Algo más o confirmo el pedido?"
```

3. En VAPI → Functions → Add Function:
   - **Name**: `create_order`
   - **URL**: `http://TU-N8N-URL/webhook/voice-order`
   - **Method**: POST

**Opción B: ElevenLabs (Solo TTS, sin STT)**

1. Registrarse en https://elevenlabs.io
2. Copiar API Key
3. En n8n, agregar credencial de ElevenLabs
4. Configurar el nodo HTTP Request con tu VOICE_ID

### 5. Conectar el HTML con n8n

Edita `voice-order-simple.html` línea ~319:

```javascript
// Cambiar esto:
// fetch('http://tu-n8n-webhook-url', {

// Por tu URL de webhook de n8n:
fetch('http://localhost:5678/webhook/voice-order', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify(orderData)
});
```

### 6. Probar el Sistema

**Test del HTML:**
```bash
# Opción 1: Python
python3 -m http.server 8000

# Opción 2: Node.js
npx http-server

# Abrir: http://localhost:8000/voice-order-simple.html
```

**Test del Webhook de n8n:**
```bash
curl -X POST http://localhost:5678/webhook/voice-order \
  -H "Content-Type: application/json" \
  -d '{
    "table": "5",
    "items": [
      {"name": "Pizza Margherita", "quantity": 1, "price": 12, "subtotal": 12}
    ],
    "total": 12,
    "timestamp": "2024-01-01T12:00:00Z"
  }'
```

---

## 📱 Opción 3: Template VAPI + PostgreSQL (MÁS COMPLETO)

Este es el template profesional de n8n para restaurantes:

### URL del Template:
https://n8n.io/workflows/5847-build-a-restaurant-voice-assistant-with-vapi-and-postgresql-for-bookings-and-orders

### Características:
- ✅ Voice AI completo (STT + TTS)
- ✅ Reservas de mesa
- ✅ Pedidos de comida
- ✅ Respuestas sobre menú/horarios
- ✅ Integración con PostgreSQL
- ✅ Confirmaciones por voz

### Instalación:
1. Ve al enlace del template
2. Click en "Use workflow"
3. Importa a tu n8n
4. Configura las credenciales VAPI y PostgreSQL
5. Activa el workflow

---

## 🎯 Comparación de Opciones

| Característica | HTML Simple | n8n + Webhook | VAPI Template |
|---------------|-------------|---------------|---------------|
| **Tiempo setup** | 5 min | 1 hora | 2 horas |
| **Reconocimiento voz** | Web API | ❌ | ✅ Telefónico |
| **Base de datos** | ❌ | ✅ | ✅ |
| **TTS (respuestas)** | ❌ | Opcional | ✅ |
| **Telefónica** | ❌ | ❌ | ✅ |
| **Notificaciones** | ❌ | ✅ Telegram | ✅ |
| **Costo** | Gratis | Gratis | $20/mes* |

*VAPI plan gratuito: 10 min/mes

---

## 🔧 Personalización

### Agregar más items al menú:

**En HTML:**
```javascript
const menu = {
    'pizza margherita': 12,
    'pizza pepperoni': 14,
    'hamburguesa': 10,      // ← Agregar aquí
    'papas fritas': 5,       // ← Agregar aquí
    // ... más items
};
```

**En VAPI:**
Editar el system prompt agregando los nuevos items.

### Cambiar idioma a inglés:

```javascript
// En voice-order-simple.html línea ~65
recognition.lang = 'en-US';  // Cambiar de 'es-ES' a 'en-US'
```

### Agregar notificación por WhatsApp:

En n8n, agregar un nodo HTTP Request apuntando a la API de WhatsApp Business.

---

## 🐛 Troubleshooting

### "El micrófono no funciona"
- Verifica que el navegador sea Chrome/Edge
- Permite permisos de micrófono
- Debe ser HTTPS o localhost

### "No reconoce los items"
- Habla claro y despacio
- Verifica que los items estén en el menú
- Revisa la consola del navegador (F12)

### "n8n no recibe los pedidos"
- Verifica que el workflow esté activo
- Revisa la URL del webhook
- Comprueba los logs de n8n

### "PostgreSQL connection error"
- Verifica credenciales
- Comprueba que PostgreSQL esté corriendo
- Revisa el firewall/puertos

---

## 📚 Recursos Adicionales

- **n8n Docs**: https://docs.n8n.io
- **VAPI Docs**: https://docs.vapi.ai
- **ElevenLabs**: https://elevenlabs.io/docs
- **Web Speech API**: https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API

---

## 🎓 Próximos Pasos

1. ✅ Testear el HTML simple (5 min)
2. ✅ Configurar PostgreSQL (10 min)
3. ✅ Instalar n8n (10 min)
4. ✅ Importar workflow (5 min)
5. 🚀 Probar en producción
6. 📈 Agregar analytics y reportes

---

## 💡 Ideas para Mejorar

- [ ] Agregar autenticación de usuarios
- [ ] Sistema de propinas
- [ ] Integración con POS existente
- [ ] Multi-idioma automático
- [ ] Reportes diarios por email
- [ ] Dashboard web para monitoreo
- [ ] Integración con Uber Eats/Rappi

---

## 🤝 Soporte

Si tienes dudas:
1. Revisa la documentación de n8n
2. Busca en el foro de n8n community
3. Prueba con datos de ejemplo primero

**¡Buena suerte con tu sistema de pedidos por voz! 🚀**