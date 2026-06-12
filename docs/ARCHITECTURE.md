# Arquitectura de Movinex - Stack Técnico

  ## Overview

  Movinex es una fintech de financiamiento de smartphones construida con una arquitectura moderna, escalable y completamente integrada.

    ```
    ┌─────────────────┐
    │  Frontend Web   │  ← cotizador.html, documentos.html (Herramientas públicas)
      │   (Cliente)     │
      └────────┬────────┘
               │
               │ HTTPS
               │
      ┌────────▼────────────────────────────┐
      │  n8n Cloud                          │  ← Orquestación, webhooks, automatización
        │ (movinex.app.n8n.cloud)             │
        │                                     │
        │  • Webhook para documentos           │
        │  • Webhook para pagos (Mercado Pago)│
        │  • Notificaciones WhatsApp           │
        │  • Lógica de automatización          │
        └────────┬────────────────────────────┘
                 │
                 ├─────────────────────────────────────────┐
                 │                                         │
        ┌────────▼──────────┐              ┌──────────────▼──────┐
        │  Supabase (PG)    │              │  WhatsApp Cloud API  │
        │                   │              │  (Meta)              │
        │  • applicants     │              │                      │
        │  • documents      │              │  WABA ID:            │
          │  • applications   │              │  1778141266929962   │
          │  • payments       │              │                      │
          │  • chats          │              │  Phone ID:           │
            │                   │              │  1196183666905090   │
            └───────────────────┘              └─────────────────────┘
                     │
                     ├─────────────────────────────────────────┐
                     │                                         │
            ┌────────▼──────────┐              ┌──────────────▼──────┐
            │  Mercado Pago API │              │  PAC (Facturación)   │
            │                   │              │                      │
            │  • Crear pagos    │              │  • Fiscalapi        │
            │  • Webhooks       │              │  • Facturapi        │
            │  • Consultar pago │              │  • Facturama        │
            │                   │              │                      │
            └───────────────────┘              └─────────────────────┘
            ```

            ## Componentes Principales

            ### 1. Frontend - Herramientas Públicas

            #### `cotizador.html`
            Calculadora interactiva de financiamiento.

            **Características:**
              - Modelo: Samsung Galaxy A07 ($19,200 MXN)
                - Enganche: 15% calculado automáticamente
                  - Planes: 26 y 52 semanas
                    - Cálculo de CAT: 974.9% (26w) / 920.3% (52w)
                      - Desglose: Comisión (3%), IVA sobre interés
                        - Respuesta móvil optimizada
                        - Colores corporativos: marino, azul, cian

                          **Flujo:**
                            1. Usuario selecciona cantidad financiada
                            2. Sistema calcula cuota semanal
                            3. Muestra desglose de costos
                            4. Botón "Solicitar ahora" redirige a documentos

                            #### `documentos.html`
                            Sistema de carga de documentación.

                            **Requisitos:**
                              - INE (anverso y reverso)
                              - Comprobante de domicilio

                              **Validaciones:**
                                - Tamaño máximo 5MB por archivo
                                - Formatos: JPG, PNG, PDF
                                  - Verificación client-side antes de envío
                                  - Preview de imágenes antes de confirmar

                                  **Flujo:**
                                    1. Usuario carga INE (2 archivos)
                                    2. Usuario carga comprobante domicilio
                                    3. Cliente valida archivos
                                    4. Botón "Enviar" → POST a webhook n8n

                                    ### 2. Backend - n8n Cloud

                                    **URL Base:** `https://movinex.app.n8n.cloud`

                                      #### Webhook: `/webhook/documents`
                                        ```
                                        POST /webhook/documents
                                      {
                                          "applicant_id": "uuid",
                                          "full_name": "string",
                                          "email": "string",
                                          "phone_whatsapp": "string",
                                          "documents": {
                                                "ine_front": "base64_encoded",
                                                "ine_back": "base64_encoded",
                                                "proof_address": "base64_encoded"
                                          }
                                      }
```

  **Acciones:**
    1. Validar estructura de datos
    2. Guardar documentos en Supabase Storage
    3. Crear registro en tabla `documents`
    4. Enviar confirmación vía WhatsApp
    5. Cambiar estado a "pending_review"

    #### Webhook: `/webhook/submit-application`
      ```
      POST /webhook/submit-application
    {
        "applicant_id": "uuid",
        "device_model": "string",
        "plan_weeks": 26 | 52,
        "down_payment": number,
        "financed_amount": number,
        "weekly_payment": number
    }
```

  **Acciones:**
    1. Validar existencia del solicitante
    2. Crear registro en tabla `applications`
    3. Iniciar cronograma de pagos en tabla `payments`
    4. Enviar resumen vía WhatsApp
    5. Cambiar estado a "active"

    #### Webhook: `/webhook/payment-confirmed`
      ```
      POST /webhook/payment-confirmed
    {
        "application_id": "uuid",
        "week_number": integer,
        "amount": number,
        "mercado_pago_id": "string"
    }
```

  **Acciones:**
    1. Actualizar registro en tabla `payments`
    2. Cambiar `paid_date` y `payment_method`
    3. Verificar si faltan pagos pendientes
    4. Si está completo, cambiar estado a "completed"

      ### 3. Base de Datos - Supabase (PostgreSQL)

      #### Tabla `applicants`
      ```sql
      id UUID PRIMARY KEY
      full_name VARCHAR
      email VARCHAR
      phone_whatsapp VARCHAR
      status VARCHAR ('pending_docs' | 'pending_review' | 'active' | 'completed' | 'rejected')
      created_at TIMESTAMP
      updated_at TIMESTAMP
      ```

      #### Tabla `documents`
      ```sql
      id UUID PRIMARY KEY
      applicant_id UUID (FK)
      doc_type VARCHAR ('ine_front' | 'ine_back' | 'proof_address')
      file_url VARCHAR (URL en Storage)
      status VARCHAR ('pending' | 'verified' | 'rejected')
      validation_notes TEXT
      uploaded_at TIMESTAMP
      verified_at TIMESTAMP
      ```

      #### Tabla `applications`
      ```sql
      id UUID PRIMARY KEY
      applicant_id UUID (FK)
      device_model VARCHAR
      plan_weeks INTEGER (26 | 52)
      down_payment NUMERIC
      financed_amount NUMERIC
      weekly_payment NUMERIC
      total_interest NUMERIC
      cat NUMERIC (974.9 | 920.3)
      status VARCHAR ('pending' | 'active' | 'completed' | 'cancelled')
      created_at TIMESTAMP
      updated_at TIMESTAMP
      ```

      #### Tabla `payments`
      ```sql
      id UUID PRIMARY KEY
      application_id UUID (FK)
      week_number INTEGER (1-26 o 1-52)
      amount NUMERIC
      due_date DATE
      paid_date DATE (nullable)
      payment_method VARCHAR ('mercado_pago' | 'manual')
      mercado_pago_id VARCHAR (nullable)
      status VARCHAR ('pending' | 'paid' | 'overdue' | 'cancelled')
      created_at TIMESTAMP
      updated_at TIMESTAMP
      ```

      #### Tabla `chats` (Futuro)
      ```sql
      id UUID PRIMARY KEY
      applicant_id UUID (FK)
      message_type VARCHAR ('inbound' | 'outbound')
      message_text TEXT
      sender VARCHAR ('applicant' | 'system')
      read BOOLEAN
      created_at TIMESTAMP
      ```

      ### 4. Integraciones Externas

      #### WhatsApp Business Cloud API
      - **WABA ID:** 1778141266929962
        - **Phone Number ID:** 1196183666905090
          - **Función:** Notificaciones en tiempo real, confirmaciones, recordatorios

            **Mensajes:** - Confirmación de documento recibido
              - Solicitud aprobada/rechazada
              - Enlace de pago semanal
              - Recordatorio de vencimiento
              - Confirmación de pago recibido

              #### Mercado Pago
              - **Función:** Procesamiento de pagos de cuotas
                - **Webhook:** Validar pago y actualizar estado

                  **Flujo:**
                    1. n8n genera enlace de pago (preferencia MP)
                    2. Envía enlace vía WhatsApp a usuario
                    3. Usuario completa pago en MP
                    4. Webhook de MP notifica a n8n
                    5. n8n actualiza tabla `payments`

                    #### PAC - Facturación CFDI 4.0
                    - **Proveedores candidatos:** Fiscalapi, Facturapi, Facturama
                      - **Función:** Generar comprobantes fiscales

                        **Fiscalidad:**
                          - Venta a plazos: Emitir CFDI de la venta completa al inicio
                            - Interés: Emitir CFDI separado por interés cobrado

                              ### 5. Panel Administrativo (Futuro)

                              **Stack:** Claude Code + React
                                **Funciones:**
                                  - Dashboard de KPIs
                                  - Gestión de aplicaciones (aprobar/rechazar)
                                  - Validación manual de documentos
                                  - Historial de pagos
                                  - Chat con usuarios
                                  - Reportes

                                  ## Flujo Completo del Usuario

                                  ```
                                  1. Usuario accede a cotizador.html
                                  2. Calcula financiamiento para Galaxy A07
                                  3. Hace click "Solicitar ahora"
                                  4. Redirige a documentos.html
                                  5. Carga INE y comprobante domicilio
                                  6. Sistema valida archivos localmente
                                  7. Envía a POST /webhook/documents (n8n)
                                  8. n8n guarda documentos en Supabase Storage
                                  9. n8n envía confirmación vía WhatsApp
                                  10. Administrador revisa documentos (panel futuro)
                                  11. Aprueba → Status "active"
                                  12. n8n genera cronograma de pagos en tabla `payments`
                                  13. Cada semana: enlace de pago vía WhatsApp
                                    14. Usuario paga en Mercado Pago
                                    15. Webhook de MP actualiza tabla `payments`
                                    16. Después de 26/52 pagos: Status "completed"
                                      17. n8n genera CFDI de venta + intereses
                                      ```

                                      ## Variables de Entorno

                                      ```bash
                                      # n8n
                                      N8N_WEBHOOK_URL=https://movinex.app.n8n.cloud/webhook

# WhatsApp
  WHATSAPP_WABA_ID=1778141266929962
  WHATSAPP_PHONE_NUMBER_ID=1196183666905090
  WHATSAPP_BUSINESS_ACCOUNT_ID=...
  WHATSAPP_ACCESS_TOKEN=...

  # Supabase
  SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=...
  SUPABASE_SERVICE_ROLE_KEY=...

  # Mercado Pago
  MERCADO_PAGO_ACCESS_TOKEN=...
  MERCADO_PAGO_WEBHOOK_TOKEN=...

  # PAC / Facturación
  PAC_API_KEY=...
  PAC_API_SECRET=...
  RFC_EMPRESA=...
  ```

  ## Roadmap Técnico

  **Fase 1 (MVP):** ✅ En desarrollo
  - Cotizador funcional
  - Carga de documentos
  - n8n webhooks
  - Notificaciones WhatsApp

  **Fase 2:** Próximas 4 semanas
    - Integración Mercado Pago completa
    - Tabla de pagos automática
    - Panel administrativo básico
    - Validación automática de documentos

    **Fase 3:** Mes 2
      - Facturación CFDI automática
      - Reportes avanzados
      - SMS como fallback de WhatsApp
      - Mobile app nativa

      **Fase 4:** Mes 3+
        - Machine learning para scoring de riesgo
        - API pública para partners
        - Expansión a otros dispositivos
        - Cobertura a Honduras / Centroamérica
