# 🚀 Movinex

> Celulares financiados al 100% en línea con entrega a domicilio en México

**Repositorio público** de herramientas, documentación y componentes de Movinex.

---

## 📦 ¿Qué incluye?

### `tools/`
- **cotizador.html** - Calculadora interactiva de financiamiento
  - Cálculo de cuotas semanales/quincenales
    - Desglose de comisiones, IVA e interés
      - Cálculo de CAT transparente
        - Respuesta móvil optimizada

        - **documentos.html** - Sistema de verificación de identidad
          - Captura de documentos (INE, domicilio)
            - Gestión de archivos cliente-side
              - Validaciones de formato e integridad

              ### `docs/`
              - **FAQ.md** - Preguntas frecuentes completas
              - **ARCHITECTURE.md** - Diagrama de arquitectura n8n + Supabase

              ---

              ## 🎯 Características técnicas

              ### Cotizador
              ```
              - Modelos: Samsung Galaxy A07
              - Planes: 26 y 52 semanas
              - Enganche: 15%
              - Comisión apertura: 3%
              - CAT: 920.3% (52 sem) / 974.9% (26 sem)
              ```

              ### Documentos
              ```
              - INE (Anverso + Reverso)
              - Comprobante de domicilio
              - Validación de tamaño y formato
              ```

              ---

              ## 🔧 Uso local

              ### Cotizador
              ```bash
              # Abrir directamente en navegador
              open tools/cotizador.html

              # O usar un servidor local
              python3 -m http.server 8000
              # http://localhost:8000/tools/cotizador.html
              ```

              ### Documentos
              ```bash
              # Mismo proceso
              open tools/documentos.html
              ```

              ---

              ## 🔗 Integración con Movinex

              Estas herramientas se integran en:

              - **Sitio web** (Wix) vía iframe o importación
              - **Flujos n8n** (webhooks de documentos, cálculos)
              - **Base de datos Supabase** (almacenamiento de solicitudes)
              - **Whatsapp API** (confirmaciones automáticas)

              ### Arquitectura backend (próximamente)
              ```
              Frontend (Wix/HTML) → n8n (orquestación) → Supabase (datos) → WhatsApp (notificaciones)
              ```

              ---

              ## 📋 Stack técnico

              **Frontend:**
              - HTML5 + CSS3 (diseño responsivo)
              - JavaScript vanilla (sin dependencias)
              - Mobile-first (tested en iOS/Android)

              **Backend (próximo):**
              - n8n Cloud para automatización
              - Supabase para base de datos + auth
              - WhatsApp Business Cloud API
              - PAC (e-invoice via Facturapi/Fiscalapi)

              ---

              ## 📞 Contacto

              - **Fundador:** Eduardo
              - **Email:** (próximamente)
              - **WhatsApp:** +52 XXX XXX XXXX
              - **Sitio:** https://movinex.mx (próximamente)

              ---

              ## 📄 Licencia

              Uso interno. Derechos reservados © 2026 Movinex.

              ---

              **Última actualización:** Junio 2026
